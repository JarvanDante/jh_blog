---
author: "karson"
title: "Golang的内存管理"
date: 2022-02-05 00:00:01
description: "Golang 的内存管理"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags: 
- golang
- categories:




---
## 思想
### 1.以空间换时间，一次缓存，多次复用
go中的堆 mheap 正是"缓存"的思想。
堆mheap的特点：
+ 对于操作系统，是用户进程中缓存的内存
+ 对于go进行内部，堆是所有对象的内存起源

### 2.多级缓存，实现无/细锁化
![/images/docImages/mem1.png](/images/docImages/mem1.png)

堆 是Go运行时中最大的临界共享资源，这意味着每次存取都要加锁，在性能层面是一件很可怕的事情。

相关概念：
1. <font color='cyan'>mheap</font> : 全局的内存起源，访问要加全局锁
2. <font color='cyan'>mcentral</font> : 每种对象大小规格（全局共划分为68种）对应的缓存，锁的粒度也仅限于同一种规格以内
3. <font color='cyan'>mcache</font> ：每个P（正是GMP中的P）持有一份的内存缓存，访问时无锁

### 3.多级规格，提高利用率
--> span大小趋势递增 
--> 分配object大小趋势递增
相关概念：
1. <font color='cyan'>page</font> ： 最小的存储单元
go借鉴操作系统分页管理的思想，每个最小的存储单元也称之为页 <font color='cyan'>**page**</font> ，但大小为 <font color='cyan'>**8KB**</font>
2. <font color='cyan'>mspan</font> : 最小的管理单元
mspan大小为page的整数倍，且从8B到80KB被划分为 68 种不同的规格，分配对象时，会根据大小映射到不同规格的mspan，从中获取空间

## 内存单元mspan
mspan中包括：next、prev指针、startAddr、allocCache（0表示被对象占用，1是空闲的 ）、pages

mspan类的源码位于 runtime/mheap.go文件中：
```go
type mspan struct {
    //标识前后节点的指针
	next *mspan     // next span in list, or nil if none
	prev *mspan     // previous span in list, or nil if none
	//起始地址
	startAddr uintptr // address of first byte of span aka s.base()
	//包含几页，页是连续的
	npages    uintptr // number of pages in span
	//标识此前的位置都已被占用
	freeindex uintptr
	//最多可以存放多少个object
	nelems uintptr // number of object in the span.
	//bitmap每个bit对应一个object块，标识该块是否已被占用
	allocCache uint64
	//标识mspan等级，包含class和noscan两部分信息
	spanclass             spanClass     // size class and noscan (uint8)
}
```

mspan等级的源码 runtime/sizeclasses.go中：

```shell
// class  bytes/obj  bytes/span  objects  tail waste  max waste  min align
//     1          8        8192     1024           0     87.50%          8
//     2         16        8192      512           0     43.75%         16
//     3         24        8192      341           8     29.24%          8
//     4         32        8192      256           0     21.88%         32
//     5         48        8192      170          32     31.52%         16
//     6         64        8192      128           0     23.44%         64
```
最大浪费率：
`（（24-77）*341 + 8）8192 = 29.24% `

代码位于 runtime/mheap.go

## 线程缓存mcache
特点：
1. mcache是每个P独有的缓存，因此交互无锁
2. mcache将每种spanClass等级的mspan各缓存一个，总数为2（noscan维度）*68（大小维度）=136
3. mcache中还有一个为对象分配器tiny allocator，用于处理小于16B对象的内存分配

mcache代码位于runtime/mcache.go:
```go
type mcache struct {
    //微对象分配器相关
    tiny       uintptr
	tinyoffset uintptr
	tinyAllocs uintptr
	
	//mcache中缓存的mspan，每种spanClass各一个
	alloc [numSpanClasses]*mspan
}
```

## 中心缓存mcentral
特点：
1. 每个mcentral对应一种spanClass
2. 每个mcentral下聚合了该spanClass下的mspan
3. mcentral下的mspan分为两个链表，分别为有空间mspan链表partial和满空间mspan链表full
4. 每个mcentral一把锁

代码位于runtime/mcentral.go:
```go
type mcentral struct {
	//对应的spanClass
	spanclass spanClass
    //有空位的mspan集合，数组长度为2是用于抗一轮GC
	partial [2]spanSet // list of spans with a free object
	//无空位的mspan集合
	full    [2]spanSet // list of spans with no free objects
}
```

## 全局堆缓存mheap
要点：
1. 对于go上层应该而言，堆是操作系统虚拟内存的抽象
2. 以页（8KB）为单位，作为最小内存存储单元
3. 负责将连续页组装成mspan
4. 全局内存基于bitMap标识其使用情况，每个bit对应一页，为0则自由，为1则已被mspan组装
5. 通过heapArena聚合页，记录了页到msapn的映射信息
6. 建立空闲页基数树索引radix tree index，辅助快速寻找空闲页
7. 是mcentral的持有者，持有所有spanClass下的mcentral，作为自身的缓存
8. 内存不够时，向操作系统申请，申请单位为heapArena

代码在 runtime/mheap.go中
```go
type mheap struct {
    //堆的全局锁
    lock mutex
    //空闲页分配器，底层是多棵基数树组成的索引，每棵树对应16GB内存空间
	pages pageAlloc
	//记录了所有的mspan，需要知道，所有mspan都是经由mheap，使用连续空闲页组装
	allspans []*mspan // all spans out there
	//heapAreana数组，64位系统下，二维数据容量为 [1][2^22]
	//每个heapArena大小64M，因此理论上，Golang堆上限为2^22*64M=256T
	arenas [1 << arenaL1Bits]*[1 << arenaL2Bits]*heapArena
	
	// 多个mcentral，总个数为spanClass的个数
	central [numSpanClasses]struct {
		mcentral mcentral
		//用于内存地址对齐
		pad      [(cpu.CacheLinePadSize - unsafe.Sizeof(mcentral{})%cpu.CacheLinePadSize) % cpu.CacheLinePadSize]byte
	}
}
```

## 空闲页索引pageAlloc
基数树数据结构的含义：
1. mheap会基于 bitMap 标识内存中各页的使用情况，<font color='cyan'>**bit位为 0 ，代表该页是空闲的，为 1 代表该页已被mspan占用**</font>。
2. 每棵基数树聚合了 <font color='cyan'>**16GB**</font> 内存空间中各页使用情况的索引信息，用于帮助mheap快速找到指定长度的连续空闲页的所在位置
3. mheap持有 <font color='cyan'>**2^14**</font> 棵基数树，因此索引全面覆盖到 <font color='cyan'>**2^14 * 16GB = 256 T**</font> 的内存空间。



## heapArena
特点：
+ 每个heapArena 包含 8192 个页，大小为 8192 * 8KB = 64 MB
+ heapArena记录了页到mspan的映射，因为GC时，通过地址偏移找到页很方便，但找到其所属的mspan不容易，因些需要通过这个映射信息进行辅助
+ heapArena是mheap向操作系统申请内存的单位（64MB）

代码位于runtime/mheap.go
```go
const pagesPerArena = 8192

type heapArena struct{
    //实现page到mspan的映射
    spans [pagesPerArena]*mspan
}
```

## 对象分配流程
分配对象的流程，不论是以下哪种方式，最终都会殊途同归步入 mallocgc 方法中

+ mew(T)
+ &T{}
+ make(xxxx)

### 分配流程总览
+ tiny微对象 (0,16B)
+ small小对象 (16B,32KB)
+ large大对象 (32KB,*)->0等级

不同类型的对象，会有着不同的分配策略，这些内容在 mallocgc 方法中都有体现.
核心流程类似于读多级缓存的过程，由上而下，每一步只要成功则直接返回. 若失败，则由下层方法兜底.
对于微对象的分配流程：
（1）从 P 专属 mcache 的 tiny 分配器取内存（无锁）
（2）根据所属的 spanClass，从 P 专属 mcache 缓存的 mspan 中取内存（无锁）
（3）根据所属的 spanClass 从对应的 mcentral 中取 mspan 填充到 mcache，然后从 mspan 中取内存（spanClass 粒度锁）
（4）根据所属的 spanClass，从 mheap 的页分配器 pageAlloc 取得足够数量空闲页组装成 mspan 填充到 mcache，然后从 mspan 中取内存（全局锁）
（5）mheap 向操作系统申请内存，更新页分配器的索引信息，然后重复（4）.
对于小对象的分配流程是跳过（1）步，执行上述流程的（2）-（5）步；
对于大对象的分配流程是跳过（1）-（3）步，执行上述流程的（4）-（5）步.

![/images/docImages/mspan.png](/images/docImages/mspan.png)

### 主干方法 mallocgc
malloc 方法主干全流程展示
代码位于 runtime/malloc.go 文件中：
```go
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
    // ...    
    // 获取 m
    mp := acquirem()
    // 获取当前 p 对应的 mcache
    c := getMCache(mp)
    var span *mspan
    var x unsafe.Pointer
    // 根据当前对象是否包含指针，标识 gc 时是否需要展开扫描
    noscan := typ == nil || typ.ptrdata == 0
    // 是否是小于 32KB 的微、小对象
    if size <= maxSmallSize {
    // 小于 16 B 且无指针，则视为微对象
        if noscan && size < maxTinySize {
        // tiny 内存块中，从 offset 往后有空闲位置
          off := c.tinyoffset
          // 如果大小为 5 ~ 8 B，size 会被调整为 8 B，此时 8 & 7 == 0，会走进此分支
          if size&7 == 0 {
                // 将 offset 补齐到 8 B 倍数的位置
                off = alignUp(off, 8)
                // 如果大小为 3 ~ 4 B，size 会被调整为 4 B，此时 4 & 3 == 0，会走进此分支  
           } else if size&3 == 0 {
           // 将 offset 补齐到 4 B 倍数的位置
                off = alignUp(off, 4)
                // 如果大小为 1 ~ 2 B，size 会被调整为 2 B，此时 2 & 1 == 0，会走进此分支  
           } else if size&1 == 0 {
            // 将 offset 补齐到 2 B 倍数的位置
                off = alignUp(off, 2)
           }
// 如果当前 tiny 内存块空间还够用，则直接分配并返回
            if off+size <= maxTinySize && c.tiny != 0 {
            // 分配空间
                x = unsafe.Pointer(c.tiny + off)
                c.tinyoffset = off + size
                c.tinyAllocs++
                mp.mallocing = 0
                releasem(mp)  
                return x
            } 
            // 分配一个新的 tiny 内存块
            span = c.alloc[tinySpanClass]    
            // 从 mCache 中获取
            v := nextFreeFast(span)        
            if v == 0 {
            // 从 mCache 中获取失败，则从 mCentral 或者 mHeap 中获取进行兜底
                v, span, shouldhelpgc = c.nextFree(tinySpanClass)
            }   
// 分配空间      
            x = unsafe.Pointer(v)
           (*[2]uint64)(x)[0] = 0
           (*[2]uint64)(x)[1] = 0
           size = maxTinySize
        } else {
          // 根据对象大小，映射到其所属的 span 的等级(0~66）
          var sizeclass uint8
          if size <= smallSizeMax-8 {
              sizeclass = size_to_class8[divRoundUp(size, smallSizeDiv)]
          } else {
              sizeclass = size_to_class128[divRoundUp(size-smallSizeMax, largeSizeDiv)]
          }        
          // 对应 span 等级下，分配给每个对象的空间大小(0~32KB)
          size = uintptr(class_to_size[sizeclass])
          // 创建 spanClass 标识，其中前 7 位对应为 span 的等级(0~66)，最后标识表示了这个对象 gc 时是否需要扫描
          spc := makeSpanClass(sizeclass, noscan) 
          // 获取 mcache 中的 span
          span = c.alloc[spc]  
          // 从 mcache 的 span 中尝试获取空间        
          v := nextFreeFast(span)
          if v == 0 {
          // mcache 分配空间失败，则通过 mcentral、mheap 兜底            
             v, span, shouldhelpgc = c.nextFree(spc)
          }     
          // 分配空间  
          x = unsafe.Pointer(v)
          // ...
       }      
       // 大于 32KB 的大对象      
   } else {
       // 从 mheap 中获取 0 号 span
       span = c.allocLarge(size, noscan)
       span.freeindex = 1
       span.allocCount = 1
       size = span.elemsize         
       // 分配空间   
        x = unsafe.Pointer(span.base())
   }  
   // ...
   return x
}
```

### 分配步骤
#### 步骤1：tiny分配
每个 P 独有的 mache 会有个微对象分配器，基于 offset 线性移动的方式对微对象进行分配，每 16B 成块，对象依据其大小，会向上取整为 2 的整数次幂进行空间补齐，然后进入分配流程.
```go
noscan := typ == nil || typ.ptrdata == 0
    // ...
        if noscan && size < maxTinySize {
        // tiny 内存块中，从 offset 往后有空闲位置
          off := c.tinyoffset
          // ...
            // 如果当前 tiny 内存块空间还够用，则直接分配并返回
            if off+size <= maxTinySize && c.tiny != 0 {
            // 分配空间
                x = unsafe.Pointer(c.tiny + off)
                c.tinyoffset = off + size
                c.tinyAllocs++
                mp.mallocing = 0
                releasem(mp)
                return x
            }
           // ...
        }
```

#### 步骤2：mcache 分配
```go
// 根据对象大小，映射到其所属的 span 的等级(0~66）
          var sizeclass uint8
          // get size class ....     
          // 对应 span 等级下，分配给每个对象的空间大小(0~32KB)
          // get span class
          spc := makeSpanClass(sizeclass, noscan) 
          // 获取 mcache 中的 span
          span = c.alloc[spc]  
          // 从 mcache 的 span 中尝试获取空间        
          v := nextFreeFast(span)
          if v == 0 {
          // mcache 分配空间失败，则通过 mcentral、mheap 兜底            
             v, span, shouldhelpgc = c.nextFree(spc)
          }     
          // 分配空间  
          x = unsafe.Pointer(v)
```
>在 mspan 中，基于 Ctz64 算法，根据 mspan.allocCache 的 bitMap 信息快速检索到空闲的 object 块，进行返回.

代码位于 runtime/malloc.go 文件中：
```go
func nextFreeFast(s *mspan) gclinkptr {
    // 通过 ctz64 算法，在 bit map 上寻找到首个 object 空位
    theBit := sys.Ctz64(s.allocCache) 
    if theBit < 64 {
        result := s.freeindex + uintptr(theBit)
        if result < s.nelems {
            freeidx := result + 1
            if freeidx%64 == 0 && freeidx != s.nelems {
                return 0
            }
            s.allocCache >>= uint(theBit + 1)
            // 偏移 freeindex 
            s.freeindex = freeidx
            s.allocCount++
            // 返回获取 object 空位的内存地址 
            return gclinkptr(result*s.elemsize + s.base())
        }
    }
    return 0
}
```

#### 步骤3：mcentral 分配
>当 mspan 无可用的 object 内存块时，会步入 mcache.nextFree 方法进行兜底.

代码位于 runtime/mcache.go 文件中：
```go
func (c *mcache) nextFree(spc spanClass) (v gclinkptr, s *mspan, shouldhelpgc bool) {
    s = c.alloc[spc]
    // ...
    // 从 mcache 的 span 中获取 object 空位的偏移量
    freeIndex := s.nextFreeIndex()
    if freeIndex == s.nelems {
        // ...
        // 倘若 mcache 中 span 已经没有空位，则调用 refill 方法从 mcentral 或者 mheap 中获取新的 span    
        c.refill(spc)
        // ...
        // 再次从替换后的 span 中获取 object 空位的偏移量
        s = c.alloc[spc]
        freeIndex = s.nextFreeIndex()
    }
    // ...
    v = gclinkptr(freeIndex*s.elemsize + s.base())
    s.allocCount++
    // ...
    return
}
```
>倘若 mcache 中，对应的 mspan 空间不足，则会在 mcache.refill 方法中，向更上层的 mcentral 乃至 mheap 获取 mspan，填充到 mache 中:

代码位于 runtime/mcache.go 文件中：
```go
func (c *mcache) refill(spc spanClass) {  
    s := c.alloc[spc]
    // ...
    // 从 mcentral 当中获取对应等级的 span
    s = mheap_.central[spc].mcentral.cacheSpan()
    // ...
    // 将新的 span 添加到 mcahe 当中
    c.alloc[spc] = s
}
```
>mcentral.cacheSpan 方法中，会加锁（spanClass 级别的 sweepLocker），分别从 partial 和 full 中尝试获取有空间的 mspan:

代码位于 runtime/mcentral.go 文件中：
```go
func (c *mcentral) cacheSpan() *mspan {
    // ...
    var sl sweepLocker    
    // ...
    sl = sweep.active.begin()
    if sl.valid {
        for ; spanBudget >= 0; spanBudget-- {
            s = c.partialUnswept(sg).pop()
            // ...
            if s, ok := sl.tryAcquire(s); ok {
                // ...
                sweep.active.end(sl)
                goto havespan
            }
            
        // 通过 sweepLock，加锁尝试从 mcentral 的非空链表 full 中获取 mspan
        for ; spanBudget >= 0; spanBudget-- {
            s = c.fullUnswept(sg).pop()
           // ...
            if s, ok := sl.tryAcquire(s); ok {
                // ...
                sweep.active.end(sl)
                goto havespan
                }
                // ...
            }
        }
        // ...
    }
    // ...


    // 执行到此处时，s 已经指向一个存在 object 空位的 mspan 了
havespan:
    // ...
    return
}
```

#### 步骤4：mheap 分配
>在 mcentral.cacheSpan 方法中，倘若从 partial 和 full 中都找不到合适的 mspan 了，则会调用 mcentral 的 grow 方法，将事态继续升级：

```go
func (c *mcentral) cacheSpan() *mspan {
    // ...
    // mcentral 中也没有可用的 mspan 了，则需要从 mheap 中获取，最终会调用 mheap_.alloc 方法
    s = c.grow()
   // ...


    // 执行到此处时，s 已经指向一个存在 object 空位的 mspan 了
havespan:
    // ...
    return
}
```

>经由 mcentral.grow 方法和 mheap.alloc 方法的周转，最终会步入 mheap.allocSpan 方法中：

```go
func (c *mcentral) grow() *mspan {
    npages := uintptr(class_to_allocnpages[c.spanclass.sizeclass()])
    size := uintptr(class_to_size[c.spanclass.sizeclass()])


    s := mheap_.alloc(npages, c.spanclass)
    // ...


    // ...
    return s
}
```

代码位于 runtime/mheap.go
```go
func (h *mheap) alloc(npages uintptr, spanclass spanClass) *mspan {
    var s *mspan
    systemstack(func() {
        // ...
        s = h.allocSpan(npages, spanAllocHeap, spanclass)
    })
    return s
}
```

代码位于 runtime/mheap.go
```go
func (h *mheap) allocSpan(npages uintptr, typ spanAllocType, spanclass spanClass) (s *mspan) {
    gp := getg()
    base, scav := uintptr(0), uintptr(0)
    
    // ...此处实际上还有一阶缓存，是从每个 P 的页缓存 pageCache 中获取空闲页组装 mspan，此处先略去了...
    
    // 加上堆全局锁
    lock(&h.lock)
    if base == 0 {
        // 通过基数树索引快速寻找满足条件的连续空闲页
        base, scav = h.pages.alloc(npages)
        // ...
    }
    
    // ...
    unlock(&h.lock)


HaveSpan:
    // 把空闲页组装成 mspan
    s.init(base, npages)
    
    // 将这批页添加到 heapArena 中，建立由页指向 mspan 的映射
    h.setSpans(s.base(), npages, s)
    // ...
    return s
}
```

#### 步骤5：向操作系统申请
>倘若 mheap 中没有足够多的空闲页了，会发起 mmap 系统调用，向操作系统申请额外的内存空间.

代码位于 runtime/mheap.go 文件的 mheap.grow 方法中：
```go
func (h *mheap) grow(npage uintptr) (uintptr, bool) {
    av, asize := h.sysAlloc(ask)
}
func (h *mheap) sysAlloc(n uintptr) (v unsafe.Pointer, size uintptr) {
       v = sysReserve(unsafe.Pointer(p), n)
}
func sysReserve(v unsafe.Pointer, n uintptr) unsafe.Pointer {
    return sysReserveOS(v, n)
}
func sysReserveOS(v unsafe.Pointer, n uintptr) unsafe.Pointer {
    p, err := mmap(v, n, _PROT_NONE, _MAP_ANON|_MAP_PRIVATE, -1, 0)
    if err != 0 {
        return nil
    }
    return p
}
```

PS:拓展：基数树寻页
核心源码位于 runtime/pagealloc.go 的 pageAlloc 方法中，要点都以在代码中给出注释：

```go
func (p *pageAlloc) find(npages uintptr) (uintptr, offAddr) {
    // 必须持有堆锁
    assertLockHeld(p.mheapLock)


    // current level.
    i := 0


    // ...
    lastSum := packPallocSum(0, 0, 0)
    lastSumIdx := -1


nextLevel:
    // 1 ~ 5 层依次遍历
    for l := 0; l < len(p.summary); l++ {
        // ...
        // 根据上一层的 index，映射到下一层的 index.
        // 映射关系示例：上层 0 -> 下层 [0~7]
        //             上层 1 -> 下层 [8~15]
        //             以此类推
        i <<= levelBits[l]
        entries := p.summary[l][i : i+entriesPerBlock]
        // ...
        // var levelBits = [summaryLevels]uint{
        //   14,3,3,3,3
        // }
        // 除第一层有 2^14 个节点外，接下来每层都只要关心 8 个 节点.
        // 由于第一层有 2^14 个节点，所以 heap 内存上限为 2^14 * 16G = 256T
        var base, size uint
        for j := j0; j < len(entries); j++ {
            sum := entries[j]
            // ...
            // 倘若当前节点对应内存空间首部即满足，直接返回结果
            s := sum.start()
            if size+s >= uint(npages) {               
                if size == 0 {
                    base = uint(j) << logMaxPages
                }             
                size += s
                break
            }
            // 倘若当前节点对应内存空间首部不满足，但是内部最长连续页满足，则到下一层节点展开搜索
            if sum.max() >= uint(npages) {               
                i += j
                lastSumIdx = i
                lastSum = sum
                continue nextLevel
            }
            // 即便内部最长连续页不满足，还可以尝试将尾部与下个节点的首部叠加，看是否满足
            if size == 0 || s < 1<<logMaxPages {
                                size = sum.end()
                base = uint(j+1)<<logMaxPages - size
                continue
            }
            // The entry is completely free, so continue the run.
            size += 1 << logMaxPages
        }
    
    // 根据 i 和 j 可以推导得到对应的内存地址，进行返回
    ci := chunkIdx(i)
    addr := chunkBase(ci) + uintptr(j)*pageSize
    // ...
    return addr, p.findMappedAddr(firstFree.base)
}
```
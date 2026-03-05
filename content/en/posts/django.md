---
title: "django框架"
date: 2020-01-25T06:40:51+09:00
description: Django是一个由Python编写的具有完整架站能力的开源Web框架。使用Django，只要很少的代码，就可以轻松地完成一个正式网站的大部分内容，并进一步开发出全功能的Web
draft: false
hideToc: false
enableToc: true
enableTocContent: false
tocPosition: inner
tags:
  - shortcode
series:
  -
categories:
  - python
image: images/feature3/django.png
---

## 一、安装

- pip 安装

{{< boxmd >}}
pip install django
{{< /boxmd >}}
安装成功后会有 <font color=#FF0000 >**django-admin**</font> 命令

- 创建新项目

{{<boxmd>}}
django-admin startproject website(项目名)
{{</boxmd>}}

- 开启项目

{{<boxmd>}}
python manage.py startapp app --不能重名
{{</boxmd>}}

- 启动项目

{{<boxmd>}}
python manage.py runserver
{{</boxmd>}}

- 指定 IP+端口 启动项目

{{<boxmd>}}
python manage.py runserver 0.0.0.0:8080
{{</boxmd>}}

![截个图](/images/feature3/screen-1.png)

## 二、迁移文件操作

### 1、创建迁移文件内容

+添加表结构 app/models.py

```python
class AssessmentInfo(models.Model):
    '''业务考核表'''
    apply_id = models.IntegerField('考核申请ID', default=0, db_index=True)
    user = models.CharField('用户名', max_length=64, default='')
    userText = models.CharField('中文名', max_length=64, default='', db_column='user_text')
    leader = models.CharField('业务考核人用户名', max_length=64, default='')
    leaderText = models.CharField('业务考核人中文名', max_length=64, default='', db_column='leader_text')
    kpi = JSONField('KPI', default=[])
    score = models.FloatField('评分', default=0)
    level = models.CharField('等级', max_length=10, default='')
    comment = models.TextField('评语', max_length=1000, default='')
    dept = models.CharField('服务部门', max_length=255, default='')
    position = models.CharField('职位', max_length=20, default='')
    entryDate = models.DateField('入场时间', blank=True, null=True, db_column='entry_date')
    # 0 - 未提交， 1- 未考核， 2 - 已考核
    status = models.IntegerField('状态', default=0)
    created = models.DateTimeField('创建时间', auto_now_add=True)
    updated = models.DateTimeField('更新时间', auto_now=True)
    STATUS_OFF = 4
    class Meta:
        db_table = 'app_assessment_info'
        index_together = [
            ['user', 'apply_id'],
            ['leader', 'apply_id'],
        ]
```

+修改表结构 app/models.py

```python
class Migration(migrations.Migration):

    dependencies = [
        ('UserManage', '0001_initial'),
    ]

    operations = [
        migrations.CreateModel(
            name='Org',
            fields=[
                ('id', models.AutoField(verbose_name='ID', serialize=False, auto_created=True, primary_key=True)),
                ('oid', models.CharField(default='', max_length=20, db_index=True)),
                ('name', models.CharField(default='', max_length=20)),
                ('level', models.IntegerField(default=0)),
                ('parent', models.IntegerField(default=0)),
                ('created', models.DateTimeField(auto_now_add=True)),
                ('updated', models.DateTimeField(auto_now=True)),
            ],
            options={
                'db_table': 'user_manage_orgs',
            },
            bases=(models.Model, common.utils.models.toDictMixin),
        ),
        migrations.AddField(
            model_name='user',
            name='entryDate',
            field=models.DateField(null=True, db_column='entry_date', blank=True),
        ),
        migrations.AddField(
            model_name='user',
            name='oid',
            field=models.CharField(default='', max_length=20),
        ),
        migrations.AddField(
            model_name='user',
            name='position',
            field=models.CharField(default='', max_length=20),
        ),
        migrations.AlterField(
            model_name='user',
            name='last_login',
            field=models.DateTimeField(null=True, blank=True),
        ),
    ]
```

### 2、执行迁移文件命令

{{<boxmd>}}
python manage.py makemigration
python manage.py migrate
{{</boxmd>}}

#### 2.1 项目添加新的数据表

##### 步骤一:

先刷新一下当前数据库的表结构，避免出现 table already exists
{{<boxmd>}}
python manage.py makemigrations
python manage.py migrate app --fake
python manage.py migrate UserManage --fake
{{</boxmd>}}

##### 步骤二:

加表结构
编写新的 models

##### 步骤三:

执行 migrate
{{<boxmd>}}
python manage.py makemigrations
python manage.py migrate app
python manage.py migrate UserManage
{{</boxmd>}}

## 三、orm 增、删、改、查操作

> ORM：Object Relational Mapping(对象关系映射)，是使用面向对象的思维来操作数据库。

### \*增加

#### 方式一 ： 类名.objects.create()

{{<boxmd>}}
models.Person.objects.create(name=username,pwd=password)
{{</boxmd>}}

#### 方式二 ： obj=models.类（属性=XX） obj.save()

{{<boxmd>}}
举子：
person_obj = models.Person(name=username,pwd=password)
person_obj.save()
{{</boxmd>}}

### \*删除

#### 方式一 ： 类名.objects.get(xxx).delete()

{{<boxmd>}}
person_obj = models.Person.objects.get(id=ids) # 获取 person 对象
person_obj.delete() # 删除数据库中的记录
{{</boxmd>}}

#### 方式二 ： 类名.objects.filter(xxx).delete()

{{<boxmd>}}
ret = models.Person.objects.filter(id=ids) # filter() 根据条件进行过滤。
ret.delete()
{{</boxmd>}}

### \*修改

#### 方式一 ：

obj =类名.objects.get(‘xxx’)
obj.zz = xx
obj.save()
{{<boxmd>}}
person_obj = models.Person.objects.get(id=id)
person_obj.name = usrname
person_obj.age = age
person_obj.birthday = bir
person_obj.save()
{{</boxmd>}}

#### 方式二 ： 类名.objects.filter(‘xxx’).update(关键字（类中的属性）=值.....)

{{<boxmd>}}
models.Person.objects.filter(id=id).update(name=usrname, age=age, birthday=bir)
{{</boxmd>}}

### \*查询

#### 方式一 ： 类名.objects.all() 获取所有记录

{{<boxmd>}}
models.Person.objects.all()
models.Person.objects.filter(is_active=True)
注意：单独查询要用 try except
try:
auth=Author.objects.get(id=1)
auth.delete()
except:
print(删除失败)
{{</boxmd>}}

## 四、路由

```python
from django.conf.urls import include, url

urlpatterns = [
    url(r'^$', Home),
    url(r'^wxlogin/$', WXLogin),
    url(r'^app/', include('app.urls')),
    url(r'^accounts/', include('UserManage.urls')),
    url(r'^os_web/partners_kpi/api/wxlogin/$', WXLogin),
    url(r'^os_web/partners_kpi/api/app/', include('app.urls')),
    url(r'^os_web/partners_kpi/api/accounts/', include('UserManage.urls')),

]
```

## 五、F 对象

用于模型类的 A 字段属性与 B 字段属性两者的比较，即操作数据库中某一列的值。
通常是对数据库中的字段值在不获取的情况下进行操作。F 对象内置在数据包 django.db.models 中，所以使用时需要提前导入

```python
from django.db.models import F
F('字段名')
```

在使用 F 对象进行查询的时候需要注意：

一个 F() 对象代表了一个 Model 的字段的值；F 对象可以在没有实际访问数据库获取数据值的情况下对字段的值进行引用。

Django 支持对 F 对象引用字段的算术运算操作，并且运算符两边可以是具体的数值或者是另一个 F 对象，下面我们通过实例进一步认识 F 对象。

```python
from django.db.models import F
from index.models import Book
#给Book所有实例价格（retail_price）涨价20元
Book.objects.all().update(retail_price=F('retail_price')+20) #获取该列所有值并加20
#利用传统的方法实现涨价20元
books = models.Book.objects.all()
for book in books:
  book.update(retail_price=book.retail_price+20)
  book.save()
```

通过上述实例可以看出，使用 F 对象相对传统的方法要简单的多。那么如何通过 F 对象实现两个字段值（列）之间的比较呢？实例如下所示：

```python
#对数据库中两个字段的值进行比较，列出哪儿些书的零售价高于定价
books = Book.objects.filter(retail_price__gt=F('price'))
for book in books:
  print(book.title, '定价:', book.price, '现价:', book.retail_price)
```

## 六、Q 对象

相比 F 对象更加复杂一点，它主要应用于包含逻辑运算的复杂查询。Q 对象把关键字参数封装在一起，并传递给 filter、exclude、get 等查询的方法。多个 Q 对象之间可以使用&或者|运算符组合（符号分别表示与和或的关系），从而产生一个新的 Q 对象。当然也可以使用~（非）运算符来取反，从而实现 NOT 查询。

Q 对象的导入方式如下所示：

```python
from django.db.models import Q
from index.models import Book
#查找c语言中文网出版的书或价格低于35的书
Book.objects.filter(Q(retail_price__lt=35)|Q(pub_id='2'))#两个Q对象是或者的逻辑关系
#查找不是c语言中文出版的书且价格低于45的书
Book.objects.filter(Q(retail_price__lt=45)&~Q(pub_id='2'))#条件1成立条件2不成立
q = Q()
if applyId:
  q &= Q(apply_id=applyId)
if status:
  q &= Q(status=status)
if applyIdList:
  q &= ~Q(apply_id__in=applyIdList)
```

Q 对象也可以与类属性的字段名组合在一起使用，但是在这种情况下，Django 规定，Q 对象必须放在 <font color="#aed581">**前面**</font>，示例如下：

```python
Book.objects.filter(Q(price__lte=100),title__icontains="p")#组合使用
<QuerySet [<Book: Book object (1)>]>
```

常见的运算符：
{{<boxmd>}}
& 与操作
| 或操作 Q(market_price**lt=50) | Q(market_price**gt=20)
~ 非操作
exact 判断，大小写敏感
contains 是否包含，大小写敏感
startwith 以什么值开头，大小写敏感
endwith 以什么值结束，大小写敏感
in 是否在哎包含的范围内 如 ： filter(status\_\_in=[1, 2])
{{</boxmd>}}

常见的比较运算符：
{{<boxmd>}}
gt 大于 如：filter(num\_\_gt=0)
gte 大于等于
lt 小于
lte 小于等于
{{</boxmd>}}

## 七、聚合查询

### 1、整表聚合

语法：
{{<boxmd>}}
聚合函数：Sum,Avg,Count,Max,Min
语法：MyModel.objects.aggregate(结果变量名=聚合函数('列'))
返回结果：结果变量名和值组成的字典
格式为：{"结果变量名":值}
{{</boxmd>}}

```python
from django.db.models import *
#或from django.db.models import Count
Book.objects.aggregate(res=Count('id'))
{'res':4}
```

### 2、分组聚合

“先分组（<font color="#aed581">**.values**</font>），再聚合(<font color="#aed581">**.annotate**</font>)”
{{<boxmd>}}
annotate
使用：QuerySet.annotate(结果变量名=聚合函数('列'))
返回值：QuerySet
{{</boxmd>}}

```python
# 先分组
query_set = Book.objects.values(‘pub’)
# 再聚合
QuerySet.annotate(名=聚合函数('列'))

querySetS = AssessmentInfo.objects.filter(q).values('level')
classNum = querySetS.annotate(res=Count('id'))
for i in classNum:
if i.get('level') == 'S':
  temp['Snum'] = i.get('res', 0)
    elif i.get('level') == 'A':
  temp['Anum'] = i.get('res', 0)
    elif i.get('level') == 'B':
  temp['Bnum'] = i.get('res', 0)
    elif i.get('level') == 'C':
  temp['Cnum'] = i.get('res', 0)
    elif i.get('level') == 'D':
  temp['Dnum'] = i.get('res', 0)
```

## 八、执行原生 sql

.query
执行原生 sql：

```python
MyModel.objects.raw(sql语句，拼接参数)
s1=Book.objects.raw('select * from bookstore_book ')
```

## 九、关系映射

### 1、一对一：OneToOneField(类名,on_delete=xxx)

级联删除：
1 models.CASCADE 同时删除
2 models.PROTESCT 不能删除
3 SET_NULL ForeignKey 为 null ，null=true
4 SET_DEFAULT 外键设置默认值

开启应用：步骤 1、2
1、python manager.py startapp xxx
2、settings.py -》INSTALLED_APPS=[
.
.
.
'XXX'
]

```python
from django.db import models
class Author(models.Model):
	name=models.CharField('姓名',max_length=11)
class Wife(models.Model):
	name=models.CharField('姓名',max_length=11)
	author=models.OneToOneField(Author,on_delete=models.CASCADE)
正向查询：谁有外键，先查谁
from .models import wife
wife=wife.objects.get(name='王夫人')
print(wife.name,'的老公是',wife.author.name)
反向查询：没有外键属性--反过来类名小写
print(author.wife.name)
```

### 2、一对多：ForeignField(类名,on_delete=xxx)

```python
from django.db import models
class Publisher(models.Model):
	name=models.CharField('姓名',max_length=11)
class Book(models.Model):
	title=models.CharField('姓名',max_length=11)
	author=models.ForeignField(Publisher,on_delete=models.CASCADE)

pub1=Publisher.objects.create(name='清大学出版社')
Book.objects.create(title=''C++,publisher=pub1)
```

## 十、cookie 和 session 的使用

### 1、cookie

以键-值对的形式
每次向服务器发请求时，都会携带 cookie 给服务器

```python
HttpResponse.set_cookie(key,value='',max_age=None,expires=None)
res=HttpResponse('set cookie is ok')
res.set_cookie('uname','gxn',3600)
return res
```

删除：

```python
HttpResponse.delete_cookie(key)
```

获取：

```python
request.COOKIES 字典(dict)
request.COOKIES.get('cookied名','默认值'')
```

### 2、session

生成独立的存储空间（格子）sessionID->借助 cookie 存储
session 要想成:必须有 cookie,session 数据很难被改，安全

#### 2.1 session 配置：

```python
settings.py中:INSTALLED_APPS=[
'django.contrib.sessions',
]
向MIDDLEWARE列表添加：
MIDDLEWARE=[
'django.contrib.sessions.middleware.SessionMiddleware',
]
```

#### 2.2 session 配置项：

settings.py 中相关配置项:

```python
SESSION_COOKIE_AGE=60*60*24*2*2
SESSION_EXPIRE_AT_BROWSER_CLOSE=True
```

浏览器关闭 session 失效

定期清理 session

```python
python manage.py clearsessions
```

每晚定时任务，可删除过期的 session 数据

#### 2.3 session 使用：

保存：

```python
request.session['KEY']=VALUE

def set_session(request):
request.session['uname']='wjb'
return HttpResponse('set sesssion is ok')
```

获取：

```python
request.session['KEY']
request.session.get('KEY','默认值')

def get_session(request):
value=request.session['uname']
return=HttpResponse('session value is %s'%(value))
```

---

ps:
<font color='fuchsia' >**迁移文件防止报错:**</font>
table exists
<font color='lime' >**解决方法:**</font>

```python
python manage.py makemigrations
python manage.py migrate app --fake
```

app 名称去 settings.py 中查看注册名称
同时删除 django_migrations 表中的数据

<font color='fuchsia' >**上传文件中文编码报错:**</font>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe6 in position 260: ordinal not in range(128)

<font color='lime' >**解决方法:**</font>

```python
import sys

# 设置中文编码
reload(sys)
sys.setdefaultencoding('utf8')
```

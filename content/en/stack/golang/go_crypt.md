---
author: "karson"
title: "对称加密算法AES/DES/3DES和非对称加密算法RSA"
date: 2022-12-04 00:00:01
description: "本文介绍了 Go 加密解密算法的实现"
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

## 对称加密算法
### 特点
加密和解密使用的是同一个密钥，数据私密性双向保证，也就是加密和解密都不能泄露密码

### 优缺点
+ 优点：加密效率高，适合大些的数据加密
+ 缺点：安全性相对非对称低

### go语言实现对称加密算法
#### AES
AES-128：key长度16 字节

AES-192：key长度24 字节

AES-256：key长度32 字节

```golang
var key []byte = []byte("hallenhallenhall")
// 填充密码长度
func PadPwd(srcByte []byte,blockSize int)  []byte {
	// 16 13       13-3 = 10
	padNum := blockSize - len(srcByte)%blockSize
	ret := bytes.Repeat([]byte{byte(padNum)}, padNum)
	srcByte = append(srcByte, ret...)
	return srcByte
}

// 加密
func AesEncoding(src string) (string,error) {
	srcByte := []byte(src)
	fmt.Println(srcByte)
	// safer
	block, err := aes.NewCipher(key)
	if err != nil {
		return src, err
	}
	// 密码填充
	NewSrcByte := PadPwd(srcByte, block.BlockSize())  //由于字节长度不够，所以要进行字节的填充
	fmt.Println(NewSrcByte)
	dst := make([]byte, len(NewSrcByte))
	block.Encrypt(dst, NewSrcByte)
	fmt.Println(dst)
	// base64编码
	pwd := base64.StdEncoding.EncodeToString(dst)
	return pwd, nil
}

// 去掉填充的部分
func UnPadPwd(dst []byte) ([]byte,error) {
	if len(dst) <= 0 {
		return dst, errors.New("长度有误")
	}
	// 去掉的长度
	unpadNum := int(dst[len(dst)-1])
	return dst[:(len(dst) - unpadNum)], nil
}

// 解密
func AesDecoding(pwd string) (string,error) {
	pwdByte := []byte(pwd)
	pwdByte, err := base64.StdEncoding.DecodeString(pwd)

	if err != nil {
		return pwd, err
	}
	block, errBlock := aes.NewCipher(key)
	if errBlock != nil {
		return pwd, errBlock
	}
	dst := make([]byte, len(pwdByte))
	block.Decrypt(dst, pwdByte) 	
	dst, _ = UnPadPwd(dst)		// 填充的要去掉
	return string(dst), nil
}
```

#### DES
DES：支持字节长度是8

```golang
// 只支持8字节的长度
var desKey = []byte("hallenha")
// 加密
func DesEncoding(src string) (string,error) {
	srcByte := []byte(src)
	block, err := des.NewCipher(desKey)
	if err != nil {
		return src, err
	}
	// 密码填充
	newSrcByte := PadPwd(srcByte, block.BlockSize())
	dst := make([]byte, len(newSrcByte))
	block.Encrypt(dst, newSrcByte)
	// base64编码
	pwd := base64.StdEncoding.EncodeToString(dst)
	return pwd, nil
}

// 解密
func DesDecoding(pwd string) (string,error) {
	pwdByte, err := base64.StdEncoding.DecodeString(pwd)
	if err != nil {
		return pwd, err
	}
	block, errBlock := des.NewCipher(desKey)
	if errBlock != nil {
		return pwd, errBlock
	}
	dst := make([]byte, len(pwdByte))
	block.Decrypt(dst, pwdByte)
	// 填充的要去掉
	dst, _ = UnPadPwd(dst)
	return string(dst), nil
}
```

#### DES(CBC模式)
des——CBC模式，key长度必须为24

```golang
// 3des的key，长度是24
var tdesKey = []byte("hallenhallenhallenhallen")
// 3des加密
func TDesEncoding(src string) (string,error) {
	srcByte := []byte(src)
	block, err := des.NewTripleDESCipher(tdesKey) // 和des的区别
	if err != nil {
		return src, err
	}
	// 密码填充
	newSrcByte := PadPwd(srcByte, block.BlockSize())
	dst := make([]byte, len(newSrcByte))
	block.Encrypt(dst, newSrcByte)
	// base64编码
	pwd := base64.StdEncoding.EncodeToString(dst)
	return pwd, nil
}


// 3des解密
func TDesDecoding(pwd string) (string,error) {
	pwdByte, err := base64.StdEncoding.DecodeString(pwd)
	if err != nil {
		return pwd, err
	}
	block, errBlock := des.NewTripleDESCipher(tdesKey) // 和des的区别
	if errBlock != nil {
		return pwd, errBlock
	}
	dst := make([]byte, len(pwdByte))
	block.Decrypt(dst, pwdByte)
	// 填充的要去掉
	dst, _ = UnPadPwd(dst)
	return string(dst), nil
}
```


## 非对称加密算法
### 特点
+ 加密和解密的密钥不同，有两个密钥（公钥和私钥）
+ 公钥：可以公开的密钥；公钥加密，私钥解密
+ 私钥：私密的密钥；私钥加密，公钥解密
+ 私密单方向保证，只要有一方不泄露就没问题


### 优缺点
+ 优点：安全性相对对称加密高
+ 缺点：加密效率低，适合小数据加密

### go语言实现非对称加密算法
消息发送方利用对方的公钥进行加密，消息接受方收到密文时使用自己的私钥进行解密

对哪一方更重要，哪一方就拿私钥

注意：
> 公钥和密钥生成的时候要有一种关联，要把密钥和公钥保存起来。


#### RSA

```golang
package main

import (
	"crypto/rand"
	"crypto/rsa"
	"crypto/x509"
	"encoding/pem"
	"fmt"
	"os"
)

// 保存生成的公钥和密钥
func SaveRsaKey(bits int) error {

	privateKey,err := rsa.GenerateKey(rand.Reader,bits)
	if err != nil {
		fmt.Println(err)
		return err
	}
	publicKey := privateKey.PublicKey

	// 使用x509标准对私钥进行编码，AsN.1编码字符串
	x509Privete := x509.MarshalPKCS1PrivateKey(privateKey)
	// 使用x509标准对公钥进行编码，AsN.1编码字符串
	x509Public := x509.MarshalPKCS1PublicKey(&publicKey)

	// 对私钥封装block 结构数据
	blockPrivate := pem.Block{Type: "private key",Bytes: x509Privete}
	// 对公钥封装block 结构数据
	blockPublic := pem.Block{Type: "public key",Bytes: x509Public}

	// 创建存放私钥的文件
	privateFile, errPri := os.Create("privateKey.pem")
	if errPri != nil {
		return errPri
	}
	defer privateFile.Close()
	pem.Encode(privateFile,&blockPrivate)

	// 创建存放公钥的文件
	publicFile, errPub := os.Create("publicKey.pem")
	if errPub != nil {
		return errPub
	}
	defer publicFile.Close()
	pem.Encode(publicFile,&blockPublic)


	return nil

}

// 加密
func RsaEncoding(src , filePath string)  ([]byte,error){

	srcByte := []byte(src)

	// 打开文件
	file,err := os.Open(filePath)
	if err != nil {
		return srcByte,err
	}

	// 获取文件信息
	fileInfo, errInfo := file.Stat()
	if errInfo != nil {
		return srcByte, errInfo
	}
	// 读取文件内容
	keyBytes := make([]byte, fileInfo.Size())
	// 读取内容到容器里面
	file.Read(keyBytes)

	// pem解码
	block,_ := pem.Decode(keyBytes)

	// x509解码
	publicKey , errPb := x509.ParsePKCS1PublicKey(block.Bytes)
	if errPb != nil {
		return srcByte, errPb
	}

	// 使用公钥对明文进行加密

	retByte, errRet := rsa.EncryptPKCS1v15(rand.Reader,publicKey, srcByte)
	if errRet != nil {
		return srcByte, errRet
	}

	return retByte,nil


}

// 解密
func RsaDecoding(srcByte []byte,filePath string) ([]byte,error) {
	// 打开文件
	file,err := os.Open(filePath)
	if err != nil {
		return srcByte,err
	}
	// 获取文件信息
	fileInfo,errInfo := file.Stat()
	if errInfo != nil {
		return srcByte,errInfo
	}
	// 读取文件内容
	keyBytes := make([]byte,fileInfo.Size())
	// 读取内容到容器里面
	_, _ = file.Read(keyBytes)
	// pem解码
	block,_ := pem.Decode(keyBytes)
	// x509解码
	privateKey ,errPb := x509.ParsePKCS1PrivateKey(block.Bytes)
	if errPb != nil {
		return keyBytes,errPb
	}
	// 进行解密
	retByte, errRet := rsa.DecryptPKCS1v15(rand.Reader,privateKey,srcByte)
	if errRet != nil {
		return srcByte,errRet
	}
	return retByte,nil
}

func main() {
	//err := SaveRsaKey(2048)
	//if err != nil {
	//	fmt.Println("KeyErr",err)
	//}
	msg, err := RsaEncoding("FanOne","publicKey.pem")
	fmt.Println("msg",msg)
	if err != nil {
		fmt.Println("err1",err)
	}
	msg2,err := RsaDecoding(msg,"privateKey.pem")
	if err != nil {
		fmt.Println("err",err)
	}
	fmt.Println("msg2",string(msg2))
}
```
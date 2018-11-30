---
layout: single
author_profile: true
title: "python模块介绍-hashlib"
date: 2018-11-30 10:30:53
# toc: true
tags:
  - python
  - hashlib
  - md5
categories:
  - python
  - python模块
---

hashlib模块是用来对字符串进行hash加密的模块（最常见的是md5），明文与密文是一一对应不变的关系；用于注册、登录时用户名、密码等加密使用。

### 函数分析

1. 共有5种加密算法: md5(),sha1(),sha224(),sha256(),sha3840(),sha512()，分别得到不同的加密密文。
2. hashlib.hexdigest()：获取加密的密文，16进制，无参数
3. hashlib.digest()：获取加密的密文，二进制，无参数
4. hashlib.copy()：复制一份当前创建的hash对象，无参数
5. update(str1,encoding('utf-8''))：更新加密的密文，得到的密文与原来的密文不相同
6. hash.name：查看当前获得的hash对象的加密算法
7. hash.digest_size：hash密钥占多少个字节
8. hash.block_size：hash数据块的大小
9. hashlib.algorithms_guaranteed：查看所有平台都支持的hash算法
10. hashlib.algorithms_available：查看所有的hash加密算法

代码实例：
```
import hashlib

def hash_fun_1(str1):
    #创建一个hahsh对象并对str1加密
    m=hashlib.md5(str1.encode('utf-8'))
    print('获取加密的密文，16进制，无参数',m.hexdigest())
    print('获取加密的密文，二进制，无参数:',m.digest())
    print('获取hash块的大小:',m.block_size)
    print('hash密钥占多少个字节:',m.digest_size)
    print('查看当前获得的hash对象的加密算法',m.name)

    #更新密文
    m.update(str1.encode('utf-8'))
    print('获取加密的密文，16进制，无参数', m.hexdigest())
    print('获取加密的密文，二进制，无参数:', m.digest())
    print('获取hash块的大小:', m.block_size)
    print('hash密钥占多少个字节:', m.digest_size)
    print('查看当前获得的hash对象的加密算法', m.name)

if __name__ == '__main__':
    hash_fun_1('mark')
```
结果：
```
获取加密的密文，16进制，无参数 ea82410c7a9991816b5eeeebe195e20a
获取加密的密文，二进制，无参数: b'\xea\x82A\x0cz\x99\x91\x81k^\xee\xeb\xe1\x95\xe2\n'
获取hash块的大小: 64
hash密钥占多少个字节: 16
查看当前获得的hash对象的加密算法 md5
获取加密的密文，16进制，无参数 ac673f4dbac79922838901b5974a419a
获取加密的密文，二进制，无参数: b'\xacg?M\xba\xc7\x99"\x83\x89\x01\xb5\x97JA\x9a'
获取hash块的大小: 64
hash密钥占多少个字节: 16
查看当前获得的hash对象的加密算法 md5
```
二、运用：
1、创建哈希对象，有两种方式：
```
m=hashlib.new('md5',b'cai')#选择md5加密函数加密字符串‘cai’
m=hashlib.md5('cai'.encode('utf-8'))#加密的另一种写法
```
2、特性用法：当需要加密的字符串过大的时候，可以使用同一个hash对象分多次加密，update(a)+update(b)=update(a+b)

举例：
```
import hashlib

m1=hashlib.md5()
m2=m1.copy()
m1.update('a'.encode('utf-8'))
m1.update('b'.encode('utf-8'))
print(m1.hexdigest())#输出密文
m2.update('ab'.encode('utf-8'))
print(m2.hexdigest())#输出另一个密文
```
运行结果：
```
187ef4436122d1cc2f40dc2b92f0eba0
187ef4436122d1cc2f40dc2b92f0eba0
```

三、hash算法加密
加密算法得到的密文不可逆，但是密文与明文之间的关系是一一对应的，这就使得解密出现了可能，使用大数据存储密文与明文对用关系，如果数据库内刚好有对应的密文，就可以找到明文完成解密，常用的解密网站：[http://www.cmd5.com/](http://www.cmd5.com/)，通过输入密文查找对于的明文。

为了增大破解的难度，一般需要对密码进行多次迭代加密，hashlib模块有一个专门的函数。

代码实例：
```
import hashlib
import binascii

#sha256为算法名称，12345678为要加密的密码
#mark指的是杂质，额外添加的东西，使得破解更难
#10 000是迭代次数，可以理解为加密次数
pwd=hashlib.pbkdf2_hmac('sha256',b'12345678',b'mark',10000)
print(binascii.hexlify(pwd).decode('utf-8'))
结果：

129d11e9ba1f3ef4e1393516d434f356363ffe68d7baca37fd1e91f0e87abe36
```
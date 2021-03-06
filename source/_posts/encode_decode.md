---
title: python 编码问题
date: 2018-12-24 11:14:47
tags: python
categories: 技术
comment: true
---

## 字节与字符

计算机存储的一切数据，文本字符、图片、视频、音频、软件都是由一串01的字节序列构成的，一个字节等于8个比特位。

而字符就是一个符号，比如一个汉字、一个英文字母、一个数字、一个标点都可以称为一个字符。

字节方便存储和网络传输，而字符用于显示，方便阅读。例如字符 "p" 存储到硬盘是一串二进制数据 `01110000`，占用一个字节的长度

## 编码与解码

先记住，任何信息，存放在存储介质中时，都是二进制流（比特流）。

- **编码过程**： 字符转换成二进制流表示的过程。
- **解码过程**： 二进制流转换成字符的过程。
- **编码规则**： 编码和解码过程中遵循的规则，例如GBK编码，UTF-8编码。

## 字符编码的来龙去脉

早期：ASCII 美国信息交换标准代码，最早的通用编码方案。发现不够用了，各自有各自的标准，如：(中国的 GBK, ANSI 美国国家标准协会认可的标准等

后面：unicode/ucs 统一字符，可能是Unicode名字好记，所以采用更为广泛。

GBK，GB2312，Latin-1，Big-5，ASCII等，它们的字符集和具体编码实现方式绑定（如GBK字符集就采用GBK编码方式），即字符和存储在介质上的二进制流一一对应。缺陷很明显，字符集扩展性差

Unicode解决了字符和二进制的对应关系，但是使用unicode表示一个字符，太浪费空间。例如：利用unicode表示“Python”需要12个字节才能表示，比原来ASCII表示增加了1倍。

由于计算机的内存比较大，并且字符串在内容中表示时也不会特别大，所以内容可以使用unicode来处理，但是存储和网络传输时一般数据都会非常多，那么增加1倍将是无法容忍的!

为了解决存储和网络传输的问题，出现了Unicode Transformation Format，学术名UTF，即：对unicode中的进行转换，以便于在存储和网络传输时可以节省空间!

- UTF-8： 使用1、2、3、4个字节表示所有字符；优先使用1个字符、无法满足则使增加一个字节，最多4个字节。英文占1个字节、欧洲语系占2个、东亚占3个，其它及特殊字符占4个
- UTF-16： 使用2、4个字节表示所有字符；优先使用2个字节，否则使用4个字节表示。
- UTF-32： 使用4个字节表示所有字符；

总结：**UTF 是为unicode编码设计的一种在存储和传输时节省空间的编码方案。**

## 乱码

**编码和解码时用了不同或者不兼容的字符编码方式。**就算同是Unicode，UTF-8和UTF-16也是不同的。

**解决乱码问题，需要把握的要点**：

- **输入某软件系统时字符所采用的编码是什么？**（从数据库或文件读取时，原来存储时的编码是什么？从网页抓取时，网页的编码是什么？从控制台输入时，控制台的编码方式是什么？）
- **软件系统中的编码方式是什么？**（原本若是UTF-8存储，GBK编码的软件系统该如何处理？）
- **输出时的编码方式是什么？**（如Python脚本处理后的字符串是Unicode编码，输出到采用GBK编码的Windows控制台时应该做什么？）

经常遇到的乱码问题：

Windows 的 cmd 是GBK，Linux 的 bash 是UTF-8。PyCharm 自带的shell不管是Windows还是Linux都是UTF-8

## py2:

python2它的默认编码是ASCII，想写中文，就必须声明文件头的coding为gbk or utf-8, 声明之后，

python2解释器仅以文件头声明的编码去解释你的代码，加载到内存后，并不会主动帮你转为unicode,也就是说，你的文件编码是utf-8,加载到内存里，你的变量字符串就也是utf-8, 这意味着什么你以utf-8编码的文件，在windows是乱码。

PY2中几个字符串相关的标识

**str**: 字符串类型与其叫**字符串**，不如叫**字节串bytes**     bytes == str 结果为`True`，用下标去访问的每一个元素都是一个**字节**。

**unicode**: 类型才是真正意义上的**字符串**，用下标去访问的每一个元素都是一个**字符**(虽然底下可能每个字符长度不同)。

```python
unicode(xxx)  unicode 方法默认的ASCII编码去解码xxx对应的编码的字符，相当于 xxx.decode()
unicode(xxx, 'utf-8') unicode 方法utf-8编码去解码xxx对应的编码的字符, 相当于 xxx.decode('utf-8')
str(xxx) 类似于 xxx.encode(‘ascii’)
```

**encode 与 decode 方法**:

encode的作用是将unicode对象（没有某种具体编码）字串按encoding参数给定的编码方式**编码**成为str对象（具有某种具体编码）字串。故而用str对象调用encode方法是错误的

decode的作用是将str对象字串（某种具体编码的）按照encoding参数给定的编码方式**解码**成unicode对象。故而用unicode对象调用decode方法是错误的

Python2 ： `str`–(decode)–> `unicode` –(encode)–> `str`

## py3:

python3 执行代码的过程

1. 解释器找到代码文件，把代码字符串按文件头定义的编码加载到内存，转成unicode
2. 把代码字符串按照语法规则进行解释，
3. 所有的变量字符都会以unicode编码声明

PY3 除了把字符串的编码改成了unicode, 还把str 和bytes 做了明确区分，

**str**: 就是unicode格式的字符

**bytes**: 就是单纯二进制字节序列

在py3里，把unicode编码后，字符串就变成了bytes格式, 就是想通过这样的方式明确的告诉你，想在py3里看字符，必须得是unicode编码，其它编码一律按bytes格式展示。 

Python3 ： `bytes` –(decode)–> `str` –(encode)–> `bytes`



看看关于 thrift 接口中文编码问题(具体看源码和测试)：

thrift 0.7.0 版本，string 传递只许是 str 对象，而不能是 unicode 对象，ttypes.py 直接传递数据给下层

thrift 0.11.0 版本，string 传递只许是 unicode 对象，而不能是 str 对象，ttypes.py 会对 unicode 统一编码成 utf-8

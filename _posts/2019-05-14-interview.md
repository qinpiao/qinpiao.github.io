---
layout: post
title:  "面试"
date:   2019-05-14 11:20:42
categories: Interview Python
tags: Python Interview
---

* content
{:toc}

记录一下面试过程中遇到的题目





## 计算机基础相关

- 一个网页中存在一个违法字符,导致将网页保存到文件时报错,请定位该字符在网页中的具体位置.
```
使用二分法,先将网页分为两部分,尝试保存,如果出现报错,继续对出现报错的部分进行二分查找,直到找到违法字符位置
```
    
- 用于解析域名到服务器IP映射的协议是什么?它是使用TCP协议还是UDP协议
```
用于解析域名到服务器映射的协议是DNS协议,DNS协议是应用层协议;
当DNS服务器之间进行域传输的时候使用TCP;
客户端查询DNS时,服务器时使用UDP传输数据;
由于UDP数据的长度是有限制的,当DNS查询超过了512字节的时候,过长的报文被截断,TC标志(截断标志)出现,这个时候会使用TCP进行发送
```

 
## 算法

- 格式化unix文件路径:

example:
```
"/a/b/../c/./d"  ==>  "/a/b/c/d" 

```
```python
def SimplifyPath(Path):
    result = ""
    for v in Path:
        if v in["..", "."]:
            continue
        result += v
    return result
```

- 格式化Json串:

example:
```
'{"a": "1", "b": {"c": {"d": "2", "e": "3"}, "f": "4"}, "h": "5"}'  ===>  '{"a": "1", "b.c.d": "2", "b.c.e": "3", "b.f": "4", "h":"5"}'

```
注意Json串中所有的元素都是字典,不存在值为list(`["a", "b", "c"]`)的情况

```python
def ConvertJson(Json):
    result, _ = ConvertDict(Json[2:], "")
    return "".join(["{", result, "}"])


def ConvertDict(subjson, prefx):
    result = ""
    while True:
        # 当遇到"}"说明字典结束,返回修正后的串,以及剩下的未编辑的串
        if "}" == subjson[0]:
            if len(subjson) > 0:
                subjson = subjson[1:]
            else:
                subjson = ""
            return result, subjson
        if "," == subjson[0]:
            result += subjson[0:3]
            subjson = subjson[3:]
            temp, subjson = ConvertPair(subjson, prefx)
            result += temp
            continue
        temp, subjson = ConvertPair(subjson, prefx)
        result += temp


def ConvertPair(subjson, prefx):
    result = ""
    temp, subjson = ConvertKey(subjson, prefx)
    # 后续的key是dict的情况
    if "{" == subjson[2]:
        subjson = subjson[4:]
        tempPrefx = temp
        temp, subjson = ConvertDict(subjson, tempPrefx)
        result += temp
    else:
        temp += '"'
        result += temp
        result += subjson[0:3]
        subjson = subjson[3:]
        while True:
            if '"' == subjson[0]:
                result += '"'
                if len(subjson) > 0:
                    subjson = subjson[1:]
                break
            result += subjson[0]
            subjson = subjson[1:]
    return result, subjson


def ConvertKey(subjson, prefix):
    item = prefix+"." if prefix else ""
    i = 1
    while True:
        if '"' == subjson[0]:
            return item, subjson[i:]
        item += subjson[0]
        subjson = subjson[1:]
        
```

- 实现2-32进制中任意进制的数转换成指定进制的数,如2进制数据1111转为8进制数据:17,要求函数输入为number, fromBase, toBase 输出为转换后的数据

```python
def BinaryConvert(number, fromBase, toBase):
    if fromBase == 10:
        s = int(number)
    else:
        s = Convert2Ten(number, fromBase)
        if toBase == 10:
            return str(s)
    result = ""
    while s > 0:
        odd = s % toBase  # 余数
        temp = s // toBase  # 商值
        if odd >= 10:
            result = chr((odd-10)+ord("A")) + result
        else:
            result = str(odd)+result
        if temp == 0:
            break
        s = temp
    return result


def Convert2Ten(number, fromBase):
    n = len(number)-1
    result = 0
    ordA = ord("A")
    for c in number:
        if ord(c) >= ordA:
            temp = (10 + (ord(c) - ordA))*math.pow(fromBase, n)
        else:
            temp = int(c)*math.pow(fromBase, n)
        result += int(temp)
        n -= 1
    return result
```
---
layout: post
title:  "Go 命令行库cobra的学习"
date:   2019-02-12 21：27:00
categories: Go
tags: Go Kubernetes 
---

* content
{:toc}

Kubernetes依赖于这个第三方库，在进行Kubernetes源码的学习前先了解下cobra

以便于后续更好的学习Kubernetes的源码




## 安装 cobra  

cobra的安装由于墙的缘故会导致有些包无法顺利下载下来，这里我们使用git clone的形式绕过

这里需要的包是sys与text 我们需要将这两个包放在 $GOPATH$/src/golang.org/x/目录下

如果你的$GOPATH下没有这个目录，可以先手动创建好$GOPATH$/src/golang.org/x/目录

1. 到$GOPATH$/src/golang.org/x/目录下执行
    ```
    git clone https://github.com/golang/sys.git
    git clone https://github.com/golang/text.git
    ```
2. 再执行`go get -u github.com/spf13/cobra/cobra`就能安装好cobra了

 
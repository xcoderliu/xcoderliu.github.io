---
layout: post
title: Ubuntu Install Boost
tag: Server
date: 2018-03-25 10:51:00.000000000 +10:00
---

1.安装依赖库：

```shell
sudo apt-get install mpi-default-dev　　#安装mpi库  
sudo apt-get install libicu-dev　　　　　#支持正则表达式的UNICODE字符集   
sudo apt-get install python-dev　　　　　#需要python的话  
sudo apt-get install libbz2-dev　　　　　#如果编译出现错误：bzlib.h: No such file or directory  
```
2.下载Boost库并解压

```shell
./bootstrap.sh
sudo ./b2
sudo ./b2 install
```

3.测试

```c++
#include<iostream>
#include<boost/bind.hpp>
using namespace std;
using namespace boost;
int fun(int x,int y){return x+y;}
int main(){
    int m=1;int n=2;
    cout<<boost::bind(fun,_1,_2)(m,n)<<endl;
    return 0;
}
```


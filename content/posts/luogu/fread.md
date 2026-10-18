---
title: "fread"
date: 2022-05-13T13:29:32+08:00
draft: false
author: "Jsxts"
tags: ["个人记录"]
slug: "fread"
original: "https://www.luogu.com.cn/article/44pvyhxy"
---
还是存一下把，免得每次都找不着。

```cpp
namespace IO{//by cyffff
    int len=0;
    char ibuf[(1<<20)+1],*iS,*iT,out[(1<<23)+1];
    #if ONLINE_JUDGE
    #define gh() (iS==iT?iT=(iS=ibuf)+fread(ibuf,1,(1<<20)+1,stdin),(iS==iT?EOF:*iS++):*iS++)
    #else
    #define gh() getchar()
    #endif
    #define reg register
    inline ll read(){
        reg char ch=gh();
        reg ll x=0;
        reg char t=0;
        while(ch<'0'||ch>'9')   t|=ch=='-',ch=gh();
        while(ch>='0'&&ch<='9') x=x*10+(ch^48),ch=gh();
        return t?-x:x;
    }
    inline void putc(char ch){
        out[len++]=ch;
    }
    template<class T>
    inline void write(T x){
        if(x<0)putc('-'),x=-x;
        if(x>9)write(x/10);
        out[len++]=x%10+48;
    }
    inline void flush(){
        fwrite(out,1,len,stdout);
        len=0;
    }
    inline char getc(){
        char ch=gh();
        while(ch<'A'||ch>'Z') ch=gh();
        return ch;
    }
}
using IO::read;
using IO::write;
using IO::flush;
using IO::getc;
using IO::putc;
```

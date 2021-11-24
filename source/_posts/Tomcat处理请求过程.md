---
title: 01.Tomcat处理请求过程
date: 2021-11-24 21:08:48
categories:
- 性能调优
tags:
- 架构师
- Tomcat
---

> 这篇文章是Tomcat底层原理解析系列的第一篇，详细介绍Tomcat是如何处理请求的

Tomcat通过**Endpoint**组件接收socket连接，接收到⼀个socket连接后会执⾏如下步骤：

1. 第⼀次从socket中获取数据到InputBuffer中，BIO对应的是InternalInputBuffer，⽗类是 

AbstractInputBuffer 

2. 然后基于InputBuffer进⾏解析数据 

3. 先解析请求⾏，把请求⽅法，请求uri，请求协议等封装到org.apache.coyote.Request对象中 

4. org.apache.coyote.Request中的属性都是MessageBytes类型，直接可以理解为字节类型，因为从 

socket中获取的数据都是字节，在解析过程中不⽤直接把字节转成字符串，并且MessageBytes虽然表 

示字节，但是它并不会真正的存储字节，还是使⽤ByteChunk基于InputBuffer中的字节数组来进⾏标 

记，标记字节数组中的哪个⼀个范围表示请求⽅法，哪个⼀个范围表示请求uri等等。 

5. 然后解析头，和解析请求⾏类似 

6. 解析完请求头后，就基于请求头来初始化⼀些参数，⽐如Connection是keepalive是close，⽐如是否 

有Content-length，并且对于的⻓度是多少等等，还包括当前请求在处理请求体时应该使⽤哪个 

InputFilter。 

7. 然后将请求交给容器 

8. 容器再将请求交给具体的servlet进⾏处理 

9. servlet在处理请求的过程中会利⽤response进⾏响应，返回数据给客户端，⼀个普通的响应过程会把 

数据先写⼊⼀个缓冲区，当调⽤flush，或者close⽅法时会把缓冲区中的内容发送给socet，下⾯有⼀ 

篇单独的⽂章讲解tomcat响应请求过程 

10. servlet处理完请求后，先会检查是否需要把响应数据发送给socket 

11. 接着看当前请求的请求体是否处理结束，是否还有剩余数据，如果有剩余数据需要把这些数据处理掉， 

以便能够获取到下⼀个请求的数据 

12. 然后回到第⼀步开始处理下⼀个请求
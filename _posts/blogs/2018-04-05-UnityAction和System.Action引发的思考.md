
---
layout: page
title: UnityAction和System.Action引发的思考
category: 
    - blogs
---


---

今天突然发现UnityAction，这是什么，怎么和Action有着类似的功能？可是如果功能如果真的一致，那unity官方提供这个不是多此一举吗。
在google的帮助下我得到了答案。(百度真是垃圾）

## UnityAction和System.Action的区别
1.首先，unity内置的.AddListener只能注册UnityAction来添加**非持久监听器**

2.如果希望在inspector面板访问委托，只能使用UnityAction，否则无法序列化。

3.注意，有人说UnityAction参数最多四个，System.Action参数远远多于UnityAction。但那是.Net环境的System.Action，这在unity环境下是不对的，因为unity用的是Mone，所以System.Action参数也最多是四个。当然如果环境变了，或许会有所不同。

## unity下的持久化监听器和常规监听器
说到UnityAction就不得不提AddListener，也就是监听器

unity下有2种监听器，即普通监听器和持久监听器
通过代码添加的是普通监听器，inspector面板下添加的是持久监听器，2种监听器有所不同。
普通监听器通过代码中的AddListener调用添加。他们添加了一个简单的零参数委托（UnityAction）回调。他们没有序列化。他们不能通过inspector访问
持久监听器指定GameObject，方法和要发送到该方法的参数。他们是序列化的。他们不能通过代码访问。

如果想访问代码中的持久化侦听器接口，以便我可以发送一些参数，而不是被迫调用一个零参数方法，建议使用lambda表达式

如果想通过代码添加持久化监听器，可以使用UnityEventTools
https://docs.unity3d.com/460/Documentation/ScriptReference/Events.UnityEventTools.html

关于持久化监听器和常规监听器这一段总结自https://forum.unity.com/threads/how-is-unityaction-different-from-nets-built-in-action.288873/


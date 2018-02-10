---
layout: page
title: unity和序列化和反序列
---




----------
[TOC]


##简介
所谓序列化就是将对象转换为字节流，反序列化则是字节流转换回对象

##C#完成序列化和反序列化
序列化
Serialize方法会利用反射机制查看对象类型含有的字段，因为之后要对这些字段序列化，同时还有分辨对象是否已被序列化（一个对象只能序列化一次，否则会死循环）。
```
/// <summary>
    /// 序列化
    /// </summary>
    /// <param name="instance"></param>
    /// <returns></returns>
    public static MemoryStream InstanceDataToMemory(object instance)
    {
        //创建一个流来容纳序列化的对象
        MemoryStream memoryStream = new MemoryStream();

        //创建一个序列化格式化器来执行序列化
        //BinaryFormatter只是C#所提供的格式化器中的一个
        BinaryFormatter binaryFormatter = new BinaryFormatter();

        //序列化对象并传入流中
        binaryFormatter.Serialize(memoryStream, instance);

        return memoryStream;
    }
```


反序列化
```
/// <summary>
    /// 反序列化
    /// </summary>
    /// <param name="stream"></param>
    /// <returns></returns>
    public static object MemoyInstanceData(Stream stream)
    {
        BinaryFormatter binaryFormatter = new BinaryFormatter();

        return binaryFormatter.Deserialize(stream);
    }
```
序列化存储为二进制文件
```
 public static void InstanceDataToFile(object instance,string dataname)
    {
       
        BinaryFormatter binaryFormatter = new BinaryFormatter();

        FileStream fileStream = new FileStream(dataname, FileMode.Create);
        try
        {
            binaryFormatter.Serialize(fileStream, instance);
        }
        catch(SerializationException e)
        {
            Debug.Log("Failed to serialize;Reason:" + e.Message);
        }
        finally
        {
            fileStream.Close();
        }
    }
```
二进制文件反序列化为原对象
```
public static T FileInstanceData<T>(string dataname)
        where T: class
    {
        
        BinaryFormatter binaryFormatter = new BinaryFormatter();

        FileStream fileStream = new FileStream(dataname, FileMode.Open);

        T instance=null;
        try
        {
            instance = (T)binaryFormatter.Deserialize(fileStream);
        }
        catch (SerializationException e)
        {
            Debug.Log("Failed to deserialize;Reason:" + e.Message);
        }
        finally
        {
            fileStream.Close();
            
        }
        return instance;
    }
```
测试
```
public class TestSer : MonoBehaviour
{
    private void Start()
    {
        Hero hero = new Hero(10, "武疮烦");

        //Stream stream = Serialization.InstanceDataToMemory(hero);

        ////数据重置
        //hero = null;
        //stream.Position = 0;

        //hero = (Hero)Serialization.MemoyInstanceData(stream);



        Serialization.InstanceDataToFile(hero, "hero.dat");
        hero = null;
        hero = Serialization.FileInstanceData<Hero>("hero.dat");

       
        if (hero!=null)
        {
            Debug.Log(hero.Level);
            Debug.Log(hero.Name);
        }

        
    }
}
```
注意：序列化和反序列需使用同一个类型的格式化器

一个流能容纳多个对象序列化之后字节块
```
public class TestSer : MonoBehaviour
{
    private void Start()
    {
        Hero hero = new Hero(10, "武疮烦");

        //Stream stream = Serialization.InstanceDataToMemory(hero);

        ////数据重置
        //hero = null;
        //stream.Position = 0;

        //hero = (Hero)Serialization.MemoyInstanceData(stream);

        Soldiercs soldiercs = new Soldiercs(0.1f, "bb");

        //Serialization.InstanceDataToFile(hero, "hero.dat");
        //hero = null;
        //hero = Serialization.FileInstanceData<Hero>("hero.dat");

        BinaryFormatter binaryFormatter = new BinaryFormatter();

        FileStream fileStream = new FileStream("HeroSoldier.dat", FileMode.Create);
        try
        {
            binaryFormatter.Serialize(fileStream, hero);
            binaryFormatter.Serialize(fileStream, soldiercs);
        }
        catch (SerializationException e)
        {
            Debug.Log("Failed to serialize;Reason:" + e.Message);
        }
        finally
        {
            fileStream.Close();
        };
        hero = null;
        soldiercs = null;


        fileStream = new FileStream("HeroSoldier.dat", FileMode.Open);

        try
        {
            hero = (Hero)binaryFormatter.Deserialize(fileStream);
            soldiercs =(Soldiercs)binaryFormatter.Deserialize(fileStream);
        }
        catch (SerializationException e)
        {
            Debug.Log("Failed to deserialize;Reason:" + e.Message);
        }
        finally
        {
            fileStream.Close();

        }
        if (hero!=null)
        {
            Debug.Log(hero.Level);
            Debug.Log(hero.Name);
        }
        if(soldiercs !=null)
        {
            Debug.Log(soldiercs.Attack);
            Debug.Log(soldiercs.Name);
        }
        
    }
}
```
##如何控制类型可序列化
那就是使用特性[Serializable]
只有使用了这个特性类，结构体类型才能被序列化
```
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
[Serializable]
public class Hero  {

    private int level;
    private string name;
    public Hero(int level,string name)
    {
        this.level = level;
        this.name = name;
    }

    public string Name
    {
        get
        {
            return name;
        }

    }

    public int Level
    {
        get
        {
            return level;
        }

    
    }
}
```
注意：
1.序列化特性只适用类，结构体，枚举，委托（枚举委托不需要显式使用[Serializable]）
2.如果一个类基类不能被序列化，那它即便添加了序列化特性也无法被序列化

默认情况下序列化会读取所有字段，无论它是private，internal等访问权限，但我们不希望有些字段被序列化，怎么办？
那就是使用System.NonSerializeldAttribute.
```
[NonSerialized]
int power;
```
假设我们希望一个字段的值不被序列化，但希望类在反序列化时能通过一个方法计算出它，又该怎么办？
使用System.Runtime.Serialization.OnDeserializedAttribute可以解决这个问题

```
int attack;
[NonSerialized]
int power;
[OnDeserialized]
private void Cal(StreamingContext context)
{
    this.power=attack*2;
}
```

这样每次反序列时都能调用这个方法为不可序列化的字段赋值

序列化方法调用步骤详解：
1.先调用[OnSerializing]特性的方法
2.接着序列化字段
反序列类同
1.先调用OnDeserializing特性标记的方法
2.然后反序列所有字段
##序列化，反序列化中上下文
OnSerializing和OnDeserializing特性方法需要使用StreamingContext参数
这边是结构体流的上下文
它包含了序列化的来源地，目的地等诸多信息

在构造格式化器实例时，Context会被初始化为null，可以修改这个Context完成深度克隆
```
/// <summary>
    /// 深度克隆
    /// </summary>
    /// <param name="o"></param>
    /// <returns></returns>
    public static object DeepClone(object o)
    {
        //构造临时内存流
        using (MemoryStream stream = new MemoryStream())
        {
            BinaryFormatter binaryFormatter = new BinaryFormatter();

            binaryFormatter.Context = new StreamingContext();

            binaryFormatter.Serialize(stream, o);

            stream.Position = 0;

            return binaryFormatter.Deserialize(stream);
        }
      
    }
```
##unity中的序列化和反序列化
Inspect显示的数据是经过序列化得到
prefab，场景文件实际是序列化后的二进制文件
Instantiate实际上是反序列化的过程
unity场景资源的垃圾回收机制（不同于C#的GC机制）是利用了序列化的机制
##unity中序列化的注意事项
1.最好public或使用了[SerializdField]
2.不是静态static的
3.不是const，readonly的
4.类型必须可被序列化
unity只能序列化类，结构体，UnityEngine.object,Array,List<T>,C#基元类型
##自定义序列化
受限unity序列化机制，有些自定义类型需要我们开发者编写可序列化中间结构或类代替自定义类型序列化和反序列化
##序列化和反序列内部过程
格式器序列化
第一步
返回MemberInfo类型对象构成的数组
第二步
序列化，返回object的数组
第三步
把对象成员和值信息放入流中
第四步
把程序集标识和类型名称放入流中
遍历一二步获得的数组，获得名称和值一并写入流中

格式化器反序列化
第一步
读取程序集标识和类型名称（如果异常会报错）
获得对象类型type
第二步
为type类型分配内存，不调用构造函数
第三步
构造MemberInfo数组
第四步
得到对象字段信息和数值
第五步
初始化字段
根据之前的数组反序列得到返回的object对象
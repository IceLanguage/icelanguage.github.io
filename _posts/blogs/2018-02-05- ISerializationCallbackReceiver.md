---
layout: page
title: 官方序列化接口 ISerializationCallbackReceiver
---

#~
----------

##简介
在unity中，序列化一直是个很头疼的问题，尽管官方支持了许多类型，但一些自定义类型和常用，比如Dictionary不能序列化，让人大呼头疼。不过幸运的是，ISerializationCallbackReceiver的出现解决了这个问题。


##官方提供的解决方案
编写一个类继承ISerializationCallbackReceiver接口，通过编写2个回调让List类型代替Dictionary参与序列化
```
using UnityEngine;
using System;
using System.Collections.Generic;

public class SerializationCallbackScript : MonoBehaviour, ISerializationCallbackReceiver
{
    public List<int> _keys = new List<int> { 3, 4, 5 };
    public List<string> _values = new List<string> { "I", "Love", "Unity" };

    //Unity doesn't know how to serialize a Dictionary
    public Dictionary<int, string>  _myDictionary = new Dictionary<int, string>();

    public void OnBeforeSerialize()
    {
        _keys.Clear();
        _values.Clear();

        foreach (var kvp in _myDictionary)
        {
            _keys.Add(kvp.Key);
            _values.Add(kvp.Value);
        }
    }

    public void OnAfterDeserialize()
    {
        _myDictionary = new Dictionary<int, string>();

        for (int i = 0; i != Math.Min(_keys.Count, _values.Count); i++)
            _myDictionary.Add(_keys[i], _values[i]);
    }

    void OnGUI()
    {
        foreach (var kvp in _myDictionary)
            GUILayout.Label("Key: " + kvp.Key + " value: " + kvp.Value);
    }
}
```

##Dictionary的序列化的泛型解决方案
但我们在unity使用Dictionary实在太频繁，不可能为每个类继承接口编写回调，一位牛人使用泛型编程为我们解决了这个问题
```
public class SerializationDictionary<TKey, TValue> : ISerializationCallbackReceiver
{
    [SerializeField]
    private List<TKey> keys;
    [SerializeField]
    private List<TValue> values;

    private Dictionary<TKey, TValue> target;
    public Dictionary<TKey, TValue> ToDictionary() { return target; }

    public SerializationDictionary(Dictionary<TKey, TValue> target)
    {
        this.target = target;
    }

    public void OnBeforeSerialize()
    {
        keys = new List<TKey>(target.Keys);
        values = new List<TValue>(target.Values);
    }

    public void OnAfterDeserialize()
    {
        var count = Math.Min(keys.Count, values.Count);
        target = new Dictionary<TKey, TValue>(count);
        for (var i = 0; i < count; ++i)
        {
            target.Add(keys[i], values[i]);
        }
    }
}
```
泛型实在太有魅力了，我等懒人必会之



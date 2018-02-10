---
layout: page
title: unity中三种数据存储方式ScriptableObject,Json,Xml和序列化
---


## 准备

我以Dictionary类型作为案例存储的数据类型，向大家介绍如何将数据序列化，如何将数据存储为ScriptableObject,Json,Xml等形式

这是我个人开发游戏所自定义的数据类型，之后的操作中会使用到
```
    [Serializable]
    public class Skill : ScriptableObject
    {
        public string sname;
        public SkillType type;

        public int power;//威力
        public int hitRate;//命中率
        public int fullPP;//pp上限
        public int curPP;//当前pp值

        //属性
        public PokeomonAttributes att;
      
    }

    public enum SkillType
    {
        物理,
        特殊,
        变化
    }
```
```
 private static Dictionary<int, Skill> allSkillDic = new Dictionary<int, Skill>();//allSkillDic是要存储的数据
```
## ScriptableObject

###简介
ScriptableObject是unity自带的一个类，继承于UnityEngine.Object，可用于制作数据配置文件，或编辑器扩展便于可视化编辑，unity会自动将这个继承ScriptableObject的类序列化，可控性弱，但用于数据的加载是效率最高的。

###编写序列化类
由于ScriptableObject的使用要求，我们需要专门编写一个脚本作为所需要存储的数据类型
```
 public class PokemonSkillsData :ScriptableObjectDictionary<int,Skill>
    {
        //需要专门为这几行代码编写一个cs文件
    }
```
由于博主自己的游戏初始化需要加载大量的数据配置文件，博主选择编写一个泛型类减少代码量

由于Dictionary不是ScriptableObject支持可序列化的数据类型，实际序列化是我们要用2个List代替Dictionary序列化
```
using System;
using System.Collections.Generic;
using UnityEngine;
[Serializable]
public class ScriptableObjectDictionary<TKey, TValue> :ScriptableObject
{
    [SerializeField]
    public List<TKey> keys = new List<TKey>();
    [SerializeField]
    public List<TValue> values = new List<TValue>();
    protected Dictionary<TKey, TValue> target;
    public Dictionary<TKey, TValue> Target
    {
        get
        {
            return target;
        }
        set
        {
            target = value;
            keys = new List<TKey>(Target.Keys);
            values = new List<TValue>(Target.Values);
        }
    }
}
```
###读写
存储
```
PokemonSkillsData serializationDictionary = ScriptableObject.CreateInstance<PokemonSkillsData>();
serializationDictionary.Target=allSkillDic;
AssetDatabase.CreateAsset(serializationDictionary, "Assets/Resources/PokemonSkills.asset");
```
运行这几行代码可以看到Resources文件下出现了PokemonSkills.asset文件

读取
```
PokemonSkillsData myData = Resources.Load<PokemonSkillsData>("PokemonSills");
allSkillDic = myData.Target;
```
##Json
###简介
SON(JavaScript Object Notation) 是一种轻量级的数据交换格式。它基于ECMAScript的一个子集。 JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯（包括C、C++、C#、Java、JavaScript、Perl、Python等）。这些特性使JSON成为理想的数据交换语言。 易于人阅读和编写，同时也易于机器解析和生成(一般用于提升网络传输速率)。
###序列化类
这里使用unity自带JsonUtility，
unity提供了ISerializationCallbackReceiver便于我们编写序列化类
```
[Serializable]
public class SerializationDictionary<TKey, TValue> : ISerializationCallbackReceiver
{
    [SerializeField]
    public List<TKey> keys = new List<TKey>();
    [SerializeField]
    public List<TValue> values = new List<TValue>();

    protected Dictionary<TKey, TValue> target;

    public Dictionary<TKey, TValue> Target
    {
        get
        {
            return target;
        }

        set
        {
            target = value;
            keys = new List<TKey>(Target.Keys);
            values = new List<TValue>(Target.Values);
        }
    }
  
    public void OnBeforeSerialize()
    {
        keys = new List<TKey>(Target.Keys);
        values = new List<TValue>(Target.Values);
    }

    public void OnAfterDeserialize()
    {
        int count = Math.Min(keys.Count, values.Count);
        target = new Dictionary<TKey, TValue>(count);
        for (int i = 0; i < count; ++i)
        {
            try
            {
                Target.Add(keys[i], values[i]);
            }
            catch(Exception e)
            {
                Debug.Log(e.Message);
            }
        }
    }


}

```
###读写
json文件的读写速度相比ScriptableObject慢一些，但比xml快，文件大小比ScriptableObject大一些，比xml小
```
SerializationDictionary<int, Skill> sDictionary = new SerializationDictionary<int, Skill>();
            sDictionary.Target = allSkillDic;

            string json = JsonUtility.ToJson(sDictionary);
            string path = Application.dataPath + "/Resources/PokemonSkills.json";
            File.WriteAllText(path, json, Encoding.UTF8);
            json = "";
            StreamReader sr = new StreamReader(path);
            json = sr.ReadToEnd();
            SerializationDictionary<int, Skill> newdic = JsonUtility.FromJson<SerializationDictionary<int, Skill>>(json);
```
###LitJson
LitJson是一个开源项目，可以简化序列化工作，不需要为每个类编写序列化类，轻松将一个object对象转换为json
注意：json里面的字段，自定义的对象一定要有
缺点：容易报错，需要转换数据类型

具体使用
去github找下下载，把LitJson文件下的cs文件集成一个类库项目，编译成dll，再通过unity的Import New Asset导入到项目中就能使用

```
            Dictionary<string, Skill> dic = new Dictionary<string, Skill>() ;
            foreach (var  o in allSkillDic)
            {
                dic.Add(o.Key + "", o.Value);
            }          
            string litjson = JsonMapper.ToJson(dic);
            string path = Application.dataPath + "/Resources/PokemonSkills.json";
            File.WriteAllText(path, litjson, Encoding.UTF8);
            litjson = "";
            StreamReader sr = new StreamReader(path);
            litjson = sr.ReadToEnd();
            Dictionary<string,Skill> dictionary= JsonMapper.ToObject<Dictionary<string, Skill>>(litjson);
```
注意：
若包含Dictionary结构，则key的类型必须是string，而不能是int类型，否则无法正确解析！
若需要小数，要使用double类型，而不能使用float，可后期在代码里再显式转换为float类型。
###Newtonsoft.Json
 Newtonsoft.Json，一款.NET中开源的Json序列化和反序列化类库(下载地址http://json.codeplex.com/，https://github.com/SaladLab/Json.Net.Unity3D/releases)。
 优点：灵活，不需要和json字段一对一
 
 使用
```
 string json = JsonConvert.SerializeObject(allSkillDic);
            string path = Application.dataPath + "/Resources/PokemonSkills.json";
            File.WriteAllText(path, json, Encoding.UTF8);
            json = "";
            StreamReader sr = new StreamReader(path);
            json = sr.ReadToEnd();
            Dictionary<string, Skill> dictionary = JsonConvert.DeserializeObject<Dictionary<string, Skill>>(json);
```

##Xml
xml解析速度相当慢，文件也相当大，可读性强，在编写编辑器扩展读写数据相当频繁地被使用。

###序列化类
这里使用系统自带的xml解析,需要编写复杂的序列化类
首先需要继承Dictionary<TKey,TValue>并编写繁琐的构造函数，这是为了  XmlSerializer对象构造时提供正确的类型
还要继承IXmlSerializable接口编写对应的方法
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.Serialization;
using System.Text;
using System.Xml.Serialization;
[Serializable]
public class XmlDictionary<TKey,TValue>:Dictionary<TKey,TValue>, IXmlSerializable
{
    XmlSerializer ks = new XmlSerializer(typeof(TKey));
    XmlSerializer vs = new XmlSerializer(typeof(TValue));

    #region 构造函数  

    public XmlDictionary()
        : base()
    {
    }

    public XmlDictionary(IDictionary<TKey, TValue> dictionary)
        : base(dictionary)
    {
    }

    public XmlDictionary(IEqualityComparer<TKey> comparer)
        : base(comparer)
    {
    }

    public XmlDictionary(int capacity)
        : base(capacity)
    {
    }

    public XmlDictionary(int capacity, IEqualityComparer<TKey> comparer)
        : base(capacity, comparer)
    {
    }

    protected XmlDictionary(SerializationInfo info, StreamingContext context)
        : base(info, context)
    {
    }

    #endregion 构造函数

    public System.Xml.Schema.XmlSchema GetSchema()
    {
        return null;
    }

    public void ReadXml(System.Xml.XmlReader reader)
    {
        bool wasEmpty = reader.IsEmptyElement;
        reader.Read();
        if (wasEmpty)
            return;

        while (reader.NodeType != System.Xml.XmlNodeType.EndElement)
        {
            reader.ReadStartElement("Item");

            reader.ReadStartElement("Key");
            TKey k = (TKey)ks.Deserialize(reader);
            reader.ReadEndElement();

            reader.ReadStartElement("Value");
            TValue v = (TValue)vs.Deserialize(reader);
            reader.ReadEndElement();

            Add(k, v);

            reader.ReadEndElement();
            reader.MoveToContent();
        }
        reader.ReadEndElement();
    }

    public void WriteXml(System.Xml.XmlWriter writer)
    {
        foreach (TKey key in Keys)
        {
            writer.WriteStartElement("Item");
            writer.WriteStartElement("Key");
            ks.Serialize(writer, key);
            writer.WriteEndElement();
            writer.WriteStartElement("Value");
            TValue value = this[key];
            vs.Serialize(writer, value);
            writer.WriteEndElement();
            writer.WriteEndElement();
        }

    }
}


```
###读写
```
 string path = Application.dataPath + "/Resources/";
            XmlDictionary<int, Skill> xmlDictionary = new XmlDictionary<int, Skill>(allSkillDic);
            using (FileStream fileStream = new FileStream(path+ "PokemonSkills.xml", FileMode.Create))
            {
                XmlSerializer xmlFormatter = new XmlSerializer(typeof(XmlDictionary<int,Skill>));
                xmlFormatter.Serialize(fileStream, xmlDictionary);
            }
            xmlDictionary.Clear();
            //反序列化获取存储的dictionary
            using (FileStream fileStream = new FileStream(path + "PokemonSkills.xml", FileMode.Open))
            {
                XmlSerializer xmlFormatter = new XmlSerializer(typeof(XmlDictionary<int, Skill>));
                xmlDictionary = (XmlDictionary<int, Skill>)xmlFormatter.Deserialize(fileStream);
            }
```

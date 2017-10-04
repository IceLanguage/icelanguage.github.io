---
layout: page
title: unity Dictionary序列化和反序列化及XML本地数据存储
category: 
    - blog
---
直接上代码

**首先是XML形式存储本地数据**
--------------

**XMLManager**类
```cs
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;
using System.Xml;
using System.Xml.Serialization;


public class XMLManager{

    /// 数据对象转换xml字符串  
    public static string SerializeObject(object pObject, System.Type ty)  
    {  
        string XmlizedString = null;  
        MemoryStream memoryStream = new MemoryStream();  
        XmlSerializer xs = new XmlSerializer(ty);  
        XmlTextWriter xmlTextWriter = new XmlTextWriter(memoryStream, Encoding.UTF8);  
        xs.Serialize(xmlTextWriter, pObject);  
        memoryStream = (MemoryStream)xmlTextWriter.BaseStream;  
        XmlizedString = UTF8ByteArrayToString(memoryStream.ToArray());  
        return XmlizedString;  
    }

    /// xml字符串转换数据对象  
    public static object DeserializeObject(string pXmlizedString, System.Type ty)  
    {  
        XmlSerializer xs = new XmlSerializer(ty);  
        MemoryStream memoryStream = new MemoryStream(StringToUTF8ByteArray(pXmlizedString));  
        XmlTextWriter xmlTextWriter = new XmlTextWriter(memoryStream, Encoding.UTF8);  
        return xs.Deserialize(memoryStream);  
    }

    /// <summary>
    /// UTF8字节数组转字符串  
    /// </summary>
    /// <param name="characters"></param>
    /// <returns></returns>
    public static string UTF8ByteArrayToString(byte[] characters)  
    {  
        UTF8Encoding encoding = new UTF8Encoding();  
        string constructedString = encoding.GetString(characters);  
        return (constructedString);  
    }

  
    /// <summary>
    /// 字符串转UTF8字节数组
    /// </summary>
    /// <param name="pXmlString"></param>
    /// <returns></returns>
    public static byte[] StringToUTF8ByteArray(System.String pXmlString)  
    {  
        UTF8Encoding encoding = new UTF8Encoding();  
        byte[] byteArray = encoding.GetBytes(pXmlString);  
        return byteArray;  
    }

    /// 创建文本文件  
    public static void CreateTextFile(string fileName, string strFileData, bool isEncryption)
    {
        StreamWriter writer;                               //写文件流  
        string strWriteFileData;
        if (isEncryption)
        {
            strWriteFileData = Encrypt(strFileData);  //是否加密处理  
        }
        else
        {
            strWriteFileData = strFileData;             //写入的文件数据  
        }

        writer = File.CreateText(fileName);
        writer.Write(strWriteFileData);
        writer.Close();                                    //关闭文件流  
    }


    /// 读取文本文件  
    public static string LoadTextFile(string fileName, bool isEncryption)
    {
        StreamReader sReader;                              //读文件流  
        string dataString;                                 //读出的数据字符串  

        sReader = File.OpenText(fileName);
        dataString = sReader.ReadToEnd();
        sReader.Close();                                   //关闭读文件流  

        if (isEncryption)
        {
            return Decrypt(dataString);                      //是否解密处理  
        }
        else
        {
            return dataString;
        }

    }
    /// 加密方法  
    /// 描述： 加密和解密采用相同的key,具体值自己填，但是必须为32位  
    public static string Encrypt(string toE)
    {
        byte[] keyArray = UTF8Encoding.UTF8.GetBytes("12348578902223367877723456789012");
        RijndaelManaged rDel = new RijndaelManaged();
        rDel.Key = keyArray;
        rDel.Mode = CipherMode.ECB;
        rDel.Padding = PaddingMode.PKCS7;
        ICryptoTransform cTransform = rDel.CreateEncryptor();
        byte[] toEncryptArray = UTF8Encoding.UTF8.GetBytes(toE);
        byte[] resultArray = cTransform.TransformFinalBlock(toEncryptArray, 0, toEncryptArray.Length);

        return Convert.ToBase64String(resultArray, 0, resultArray.Length);
    }

    /// 解密方法  
    /// 描述： 加密和解密采用相同的key,具体值自己填，但是必须为32位  
    public static string Decrypt(string toD)
    {
        byte[] keyArray = UTF8Encoding.UTF8.GetBytes("12348578902223367877723456789012");
        RijndaelManaged rDel = new RijndaelManaged();
        rDel.Key = keyArray;
        rDel.Mode = CipherMode.ECB;
        rDel.Padding = PaddingMode.PKCS7;
        ICryptoTransform cTransform = rDel.CreateDecryptor();
        byte[] toEncryptArray = Convert.FromBase64String(toD);
        byte[] resultArray = cTransform.TransformFinalBlock(toEncryptArray, 0, toEncryptArray.Length);

        return UTF8Encoding.UTF8.GetString(resultArray);
    }

    /// <summary>
    /// 写入数据
    /// </summary>
    /// <param name="pObject"></param>
    /// <param name="ty"></param>
    /// <param name="dataPath"></param>
    public static void XmlLocalStorageWrite(object pObject, System.Type ty, string dataPath)
    {
        //存储数据  
        string s = SerializeObject(pObject, ty);
        //创建XML文件且写入数据  
        CreateTextFile(dataPath, s, false);
    }


    
}

```
相比PlayerPrefs，能存储自定义数据结构的数据
如

```cs
public class User
{
	private string name;
	private string pas;
}
//user[],Link<user>等都可以存储在本地,但Dictionary<int,user>不行
```

**xml存储和反序列化获取方法**
--------------

```cs
 public static string dataPathSave = "F:/unityLearning/Assets/Save/";//路径
 
  public static List<user> userList = new List<user>();

 XMLManager.XmlLocalStorageWrite(userList, typeof(List<user>), dataPathSave + "user");//存储

//反序列化获取
 string strTemp = XMLManager.LoadTextFile(dataPathSave + "user", false);
 List<user> List = XMLManager.DeserializeObject(strTemp, typeof(List<user>)) as List<user>;
```

**Dictionary序列化和反序列化**
之前提到dictionary不能成功，所有有了下面的方法
**DictionaryXML**
```cs
using System;
using System.Collections.Generic;
using System.Runtime.Serialization;
using System.Xml.Serialization;

public class DictionaryXML
{
    [Serializable]
    public class SerializableDictionary<TKey, TValue> : Dictionary<TKey, TValue>, IXmlSerializable
    {
        #region 构造函数  

        public SerializableDictionary()
            : base()
        {
        }

        public SerializableDictionary(IDictionary<TKey, TValue> dictionary)
            : base(dictionary)
        {
        }

        public SerializableDictionary(IEqualityComparer<TKey> comparer)
            : base(comparer)
        {
        }

        public SerializableDictionary(int capacity)
            : base(capacity)
        {
        }

        public SerializableDictionary(int capacity, IEqualityComparer<TKey> comparer)
            : base(capacity, comparer)
        {
        }

        protected SerializableDictionary(SerializationInfo info, StreamingContext context)
            : base(info, context)
        {
        }

        #endregion 构造函数  

        #region IXmlSerializable Members  

        public System.Xml.Schema.XmlSchema GetSchema()
        {
            return null;
        }

        /// <summary>  
        /// 从对象的 XML 表示形式生成该对象  
        /// </summary>  
        /// <param name="reader"></param>  
        public void ReadXml(System.Xml.XmlReader reader)
        {
            XmlSerializer keySerializer = new XmlSerializer(typeof(TKey));
            XmlSerializer valueSerializer = new XmlSerializer(typeof(TValue));
            bool wasEmpty = reader.IsEmptyElement;
            reader.Read();
            if (wasEmpty)
                return;
            while (reader.NodeType != System.Xml.XmlNodeType.EndElement)
            {
                reader.ReadStartElement("item");
                reader.ReadStartElement("key");
                TKey key = (TKey)keySerializer.Deserialize(reader);
                reader.ReadEndElement();
                reader.ReadStartElement("value");
                TValue value = (TValue)valueSerializer.Deserialize(reader);
                reader.ReadEndElement();
                this.Add(key, value);
                reader.ReadEndElement();
                reader.MoveToContent();
            }
            reader.ReadEndElement();
        }

        /**/

        /// <summary>  
        /// 将对象转换为其 XML 表示形式  
        /// </summary>  
        /// <param name="writer"></param>  
        public void WriteXml(System.Xml.XmlWriter writer)
        {
            XmlSerializer keySerializer = new XmlSerializer(typeof(TKey));
            XmlSerializer valueSerializer = new XmlSerializer(typeof(TValue));
            foreach (TKey key in this.Keys)
            {
                writer.WriteStartElement("item");
                writer.WriteStartElement("key");
                keySerializer.Serialize(writer, key);
                writer.WriteEndElement();
                writer.WriteStartElement("value");
                TValue value = this[key];
                valueSerializer.Serialize(writer, value);
                writer.WriteEndElement();
                writer.WriteEndElement();
            }
        }
        #endregion IXmlSerializable Members  
    }


}


```

**调用**

```cs
 public static DictionaryXML.SerializableDictionary<string, user> Dic = new DictionaryXML.SerializableDictionary<string, user>();//不能是 public static Dictionary<string, user> Dic = new Dictionary<string, user>();

//存储
 using (FileStream fileStream = new FileStream(dataPathSave + "pokraceDic", FileMode.Create))
        {
            XmlSerializer xmlFormatter = new XmlSerializer(typeof(DictionaryXML.SerializableDictionary<string, user>));
            xmlFormatter.Serialize(fileStream,Dic);
        }


//反序列化获取存储的dictionary
using (FileStream fileStream = new FileStream(dataPathSave + "user", FileMode.Open))
                {
                    XmlSerializer xmlFormatter = new XmlSerializer(typeof(DictionaryXML.SerializableDictionary<string, user>));
                    Dic = (DictionaryXML.SerializableDictionary<string, user>)xmlFormatter.Deserialize(fileStream);
                }
```
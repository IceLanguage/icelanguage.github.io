---
layout: page
title: Unity单例模式泛型类
    - blogs
---

```
public class UnitySingleton<T> : MonoBehaviour  
    where T : Component  
{  
    private static T _instance;  
    public static T Instance {  
        get {  
            if (_instance == null) {  
                instance = FindObjectOfType (typeof(T)) as T;  
                if (_instance == null) {  
                    GameObject obj = new GameObject ();  
                    obj.hideFlags = HideFlags.HideAndDontSave;//隐藏实例化的new game object，下同  
                    _instance = obj.AddComponent (typeof(T));  
                }  
            }  
            return _instance;  
        }  
    }  
}
```
以后继承单例模式，直接继承这个类就可以了，需要的话可以根据需求更改模板类

转载自http://blog.csdn.net/ycl295644/article/details/49487361
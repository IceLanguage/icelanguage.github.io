---
layout: page
title: unity编辑器扩展篇-视图拓展基础
---

为了更方便的编辑，有时候需要在拓展视图，在编辑游戏时在看到一些东西

```
using UnityEditor;
using UnityEngine;
 
//自定义Tset脚本
[CustomEditor(typeof(Test))] 
//请继承Editor
public class MyEditor : Editor 
{
 
	 void OnSceneGUI() 
	{
		 //得到test脚本的对象
		 Test test = (Test) target;
 
		 //绘制文本框
		 Handles.Label(test.transform.position + Vector3.up*2,
                    test.transform.name +" : "+ test.transform.position.ToString() );
 
	//开始绘制GUI
	Handles.BeginGUI();
 
    //规定GUI显示区域
    GUILayout.BeginArea(new Rect(100, 100, 100, 100));
 
    //GUI绘制一个按钮
    if(GUILayout.Button("这是一个按钮!"))
	{
		Debug.Log("test");		
	}
	//GUI绘制文本框
	GUILayout.Label("我在编辑Scene视图");	
 
    GUILayout.EndArea();
 
	Handles.EndGUI();
	}
 
}
```
这个文件放在Editor下就能在Scene视图中看到一些test.cs的数据和你要显示的UI了

转载在http://www.xuanyusong.com/archives/2303
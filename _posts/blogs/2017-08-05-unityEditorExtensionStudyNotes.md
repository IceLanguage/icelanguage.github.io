---
layout: page
title: unity编辑器扩展学习笔记
category: 
    - blogs
---

注意事项：

 1. 必须在Assets/Editor文件下创建脚本
 2. 必须static，这样才能通过类名调用，否则需要通过对象调用 
 3. 需要using UnityEngine;


（一）MenuItem添加菜单按钮
```cs
[MenuItem("tools/test")]
    static void test()
    {
        Debug.Log("test");
    }
```
效果
![这里写图片描述](http://img.blog.csdn.net/20170805014819415?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
菜单栏出现tools，tools下有test，点击test将执行test（）

（二）菜单优先级

```java
[MenuItem("tools/test", false, 444)]
    static void test()
    {
        Debug.Log("test");
    }

    [MenuItem("tools/test", false, 44)]
    static void test3()
    {
        Debug.Log("test3");
    }

    [MenuItem("tools/test2", false, 3444)]
    static void test2()
    {
        Debug.Log("test2");
    }
```

效果
![这里写图片描述](http://img.blog.csdn.net/20170805020805073?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

第三个参数为priority决定了优先级，越小同一级下越靠前

（三）project右键菜单

```Cpp
[MenuItem("Assets/test2", false, 3444)]
    static void test2()
    {
        Debug.Log("test2");
    }
```
效果
![这里写图片描述](http://img.blog.csdn.net/20170805021255751?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170805021308407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQyNDQzMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


（四）inspect某组件区域右键菜单

```pytghon
 [MenuItem("CONTEXT/PlayerHealth/InitHealthAndSpeed")]// CONTEXT 组件名 按钮名
    static void InitHealthAndSpeed()
    {
        Debug.Log("Init");

    }
```
必须是   “CONTEXT/组件名/菜单名”


以下方法可获取到操作的组件
```lua
[MenuItem("CONTEXT/PlayerHealth/InitHealthAndSpeed")]// CONTEXT 组件名 按钮名
    static void InitHealthAndSpeed( MenuCommand cmd )//menucommand是当前正在操作的组件
    {
        //Debug.Log(cmd.context.GetType().FullName);
        CompleteProject.PlayerHealth health = cmd.context as CompleteProject.PlayerHealth;
        health.startingHealth = 200;
        health.flashSpeed = 10;
        Debug.Log("Init");
    }
```
（五）获取选择物体，进行操作

```ruby
 [MenuItem("GameObject/my delete", true, 11)]
    static bool MyDeleteValidate()
    {
	    Debug.Log(Selection.activeGameObject.name );//Selection.activeGameObject是我们第一个选择的游戏物体 
        if (Selection.objects.Length > 0)//Selection.objects获得选择所有物体
            return true;
        else
            return false;
    }
```

删除选择物体

```perl
[MenuItem("GameObject/my delete", false, 11)]
    static void Mydelete()
    {
        foreach (Object o in Selection.objects)
        {
            //GameObject.DestroyImmediate(o);//无法撤销
            Undo.DestroyObjectImmediate(o);//利用Undo进行的删除操作 是可以撤销的
        }
        //需要把删除操作注册到 操作记录里面
    }
```
（六）快捷键

```php
[MenuItem("Tools/test2 %q",false,100)]//快捷键为ctrl+q
    static void Test2()
    {
        Debug.Log("Test2");
    }
```
 注意事项
 %=ctrl 
 #=shift
  &=alt
记得加空格

（七）菜单是否可用

```net
[MenuItem("GameObject/my delete", true, 11)]//true时无法点击
    static bool MyDeleteValidate()//验证方法，会最先执行该方法，返回true才能执行其他方法
    {
        if (Selection.objects.Length > 0)
            return true;
        else
            return false;
    }
```
（八）ContextMenuItem和ContextMenu
不是在Editor下，而是在组件里
ContextMenu组件区域右键点击

```javascript
[ContextMenu("SetColor")]
        void SetColor()
        {
            flashColour = Color.green;
        }
```
ContextMenuItem是给某组件某个属性右键点击添加菜单
```vb
[ContextMenuItem("Add Hp","AddHp")] //Add Hp为菜单名称
void AddHp()
        {
            startingHealth += 20;
        }
```
（八）对话框
创建对话框
```css
public class EnemyChange : ScriptableWizard {//必须继承ScriptableWizard

    [MenuItem("Tools/CreateWizard")]
    static void CreateWizard()
    {
        ScriptableWizard.DisplayWizard<EnemyChange>("统一修改敌人","Change And Close","Change");//Change And Close为按钮名字
    }
```

检测对话框按钮点击

```html
 //检测create按钮的点击
    void OnWizardCreate()
    {

    }
```

```ruby
//当前对话框字段值修改的时候会被调用
    void OnWizardUpdate()
    {
```
对话框第二个按钮

```cs
ScriptableWizard.DisplayWizard<EnemyChange>("统一修改敌人","Change And Close","Change");//change为第二个button
```
第二个button的点击事件，other button 不同于第一个button，点击后不会自动关闭对话框
```cs
void OnWizardOtherButton()
```

（九）记录操作，以便撤销

```cs
  Undo.RecordObject(hp, "change health and speed");
  
```
（十）显示提示信息

```cs
void OnWizardCreate()
    {

        ShowNotification(new GUIContent(Selection.gameObjects.Length + "个游戏物体的值被修改了"));
    }
```
（十一）保存数据

```cs
  EditorPrefs.SetInt(changeStartHealthValueKey, changeStartHealthValue);//第一个参数是key，第二个是保存的值
```
（十二）进度条

```cs
  void OnWizardCreate()
    {
        GameObject[] enemyPrefabs = Selection.gameObjects;
        EditorUtility.DisplayProgressBar("进度", "0/" + enemyPrefabs.Length + " 完成修改值", 0);
        int count = 0;
        foreach (GameObject go in enemyPrefabs)
        {
            CompleteProject.EnemyHealth hp = go.GetComponent<CompleteProject.EnemyHealth>();
            Undo.RecordObject(hp, "change health and speed");
            hp.startingHealth += changeStartHealthValue;
            hp.sinkSpeed += changeSinkSpeedValue;
            count++;
            EditorUtility.DisplayProgressBar("进度", count+"/" + enemyPrefabs.Length + " 完成修改值", (float)count/enemyPrefabs.Length);
        }
```

```cs
  EditorUtility.DisplayProgressBar("进度", "0/" + enemyPrefabs.Length + " 完成修改值", 0);
  
```
（十三）自定义窗口
必须继承EditorWindow
```c
public class MyWindow : EditorWindow {

    [MenuItem("Window/show mywindow")]
    static void ShowMyWindow()
    {
        MyWindow window= EditorWindow.GetWindow<MyWindow>();
        window.Show();
    }

    private string name="";
    void OnGUI()
    {
        GUILayout.Label("这是我的窗口");
        name = GUILayout.TextField(name);
        if (GUILayout.Button("创建"))
        {
            GameObject go = new GameObject(name);
            Undo.RegisterCreatedObjectUndo(go, "create gameobject");//注册到操作记录
        }
    }
```
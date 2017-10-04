---
layout: page
title: 场景加载环形进度条
category: 
    - blog
---
**UIManager**
负责UI

```cs
public class UIManager : MonoBehaviour {

    public static UIManager _instance;

    public GameObject ProgressBar;
    private void Awake()
    {
        _instance = this;
    }
    public void ShowProgressBar()//显示进度条
    {
        ProgressBar.SetActive(true);
    }
}
```
**LoadScene**
负责场景加载

```cs
public class LoadScene : MonoBehaviour {

    public static LoadScene _instance;
    public AsyncOperation async;

    void Awake()
    {
        _instance = this;
    }

    public void LoadSceneByName(string name)
    {
        UIManager._instance.ShowProgressBar();
        
        StartCoroutine(loadScene(name));
        //利用协程完成场景加载和进度条显示的同步执行
    }
 
    IEnumerator loadScene(string sceneName)
    {
        async = SceneManager.LoadSceneAsync(sceneName);
        //获取场景加载进度
        
        async.allowSceneActivation = false;
        //控制正在加载的场景不显示
        
        yield return async;
    }

}
```


**circleProcess**
环形进度条控件逻辑
```cs
using UnityEngine;
using System.Collections;
using UnityEngine.UI;

public class circleProcess : MonoBehaviour {
	
	[SerializeField]
	float speed;//进度增加速度
	
	[SerializeField]
	Transform process;
	
	[SerializeField]
	Transform indicator;
	
	public int targetProcess{ get; set;}//当前进度
	private float currentAmout = 0;//进度条显示进度
    private AsyncOperation async;

	void Update () {
        async = LoadScene._instance.async;//获取进度
        if (async == null)
        {
            return;
        }
        //async.progress只能到0.9f,之后直接加载新场景
        if (async.progress < 0.9f)
        {
            targetProcess = (int)async.progress * 100;
        }
        else
        {
            targetProcess =90;
        }
        if (currentAmout == 90)
        {
            LoadScene._instance.async.allowSceneActivation = true;
        }
        //显示进度信息
        if (currentAmout <= targetProcess) {

            currentAmout = async.progress * 100;
            currentAmout += speed;
			indicator.GetComponent<Text>().text = ((int)currentAmout).ToString() + "%";//显示的进度数据
			process.GetComponent<Image>().fillAmount = currentAmout/100.0f;//环形进度显示
		}
    }
	
	
	public void SetTargetProcess(int target)
	{
		if(target >= 0 && target <= 100)
			targetProcess = target;
	}

}

```
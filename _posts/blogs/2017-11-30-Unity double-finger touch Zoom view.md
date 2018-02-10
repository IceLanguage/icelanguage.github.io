---
layout: page
title: Unity双指触控缩放视野
---



Input.touchCount为屏幕触摸的数量（2个手指）
我们通过双指移动完成对视野缩放，所以屏幕上需要至少2个手指并且手指在移动

```
if ((Input.touchCount  ==2) && (Input.GetTouch(0).phase == TouchPhase.Moved || Input.GetTouch(1).phase == TouchPhase.Moved))
        {
        }
```
说明：
Input.touchCount为屏幕上的触摸数（也就是2个手指）
Input.GetTouch(0)，Input.GetTouch(1)分别指向2个手指
Input.GetTouch(1).phase == TouchPhase.Moved是手指在移动的意思


接下来通过2手指间距的变换来缩放屏幕
先获取手指间距
category: 
    - blogs

```
Touch touch1 = Input.GetTouch(0);
            Touch touch2 = Input.GetTouch(1);

            DoubleTouchCurrDis = Vector2.Distance(touch1.position, touch2.position);
```
通过手指间距的变化，判断是缩小视野，还是放大视野

```
//是否缩放
private bool IsZoom = false;
//当前双指触控间距
private float DoubleTouchCurrDis;
//过去双指触控间距
private float DoubleTouchLastDis;

if (!IsZoom )
{
    DoubleTouchLastDis = DoubleTouchCurrDis;
    IsZoom = true;
}

float distance = DoubleTouchCurrDis - DoubleTouchLastDis;
DoubleTouchLastDis = DoubleTouchCurrDis;
```
我们通过distance这一数据完成缩放
比如这样

```
Translate( new Vector3( 0 , 0 , distance*Time.deltaTime*3 ) );//移动摄像机
```
通过distance具体的数值变化控制缩放
不过由于博主手残，选择了固定变化
如下

```
  mainCamera.updistance += (distance > 0 ? 1 : -1) * 1 ;//更改了摄像头的高度
```
这样不用刻意通过手指来控制缩放视野的比例，只要有缩放的手势，就能缓慢缩放视野

完整代码如下
```
 //是否缩放
    private bool IsZoom = false;
    //当前双指触控间距
    private float DoubleTouchCurrDis;
    //过去双指触控间距
    private float DoubleTouchLastDis;



```

update下
```
if ((Input.touchCount > 1) && (Input.GetTouch(0).phase == TouchPhase.Moved || Input.GetTouch(1).phase == TouchPhase.Moved))
        {
            Touch touch1 = Input.GetTouch(0);
            Touch touch2 = Input.GetTouch(1);

            DoubleTouchCurrDis = Vector2.Distance(touch1.position, touch2.position);

            if (!IsZoom )
            {
                DoubleTouchLastDis = DoubleTouchCurrDis;
                IsZoom = true;
            }

            float distance = DoubleTouchCurrDis - DoubleTouchLastDis;

            mainCamera.updistance += (distance > 0 ? 1 : -1) * 1 ;//更改了摄像头的高度

            DoubleTouchLastDis = DoubleTouchCurrDis;
        }
```
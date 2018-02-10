---
layout: page
title: unity围绕主角的摄像头视角控制
---

~~~cs
public GameObject player;
    //自身旋转的速度  
    private static float ax = 0;
    private static float ay = 0;
    private void Update()
    {
        LookPlayerRotate();
    }

    void LookPlayerRotate()
    {
        ay += Input.GetAxis("Mouse Y") * 5;
        ax += Input.GetAxis("Mouse X") * 5;
        //获取鼠标移动
        
        ay = Mathf.Clamp(ay, -60, 0);
        //现在摄像头的旋转角度，防止摄像头越界，导致角色道理等
        
        Quaternion q = Quaternion.Euler(ay, ax, 0);
        //通过鼠标移动控制摄像头的旋转
        
        Vector3 direction = q * Vector3.forward;
        //获取主角摄像头间的向量
		
		//direction*30为摄像头与主角的距离
		//Vector3.up * 10为摄像头距离主角中心点的高度
        transform.position = player.transform.position + direction*30 + Vector3.up * 10;
        

        transform.LookAt(player.transform.position);
        //摄像头视角设置为主角
    }
~~~
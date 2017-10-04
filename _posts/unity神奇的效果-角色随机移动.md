```
 Vector3 velocity = Vector3.zero;//速度
 float speed = 5;
private void Update()
    {

        if (Random.value < 0.01f) target = transform.position + Quaternion.Euler(0, Random.value * 360, 0) * Vector3.right * 10;//随机指定目标
        Vector3 direct = target - transform.position;
        direct.y = 0;//防止y方向移动
        if (direct.sqrMagnitude > 1)
        {
            transform.rotation = Quaternion.LookRotation(direct);//改变朝向
            Velocity=direct.normalized * speed / 3;
        }
 


        velocity -= GetComponent<Rigidbody>().velocity;
        velocity.y = 0;
        
        //速度过大时减速
        if (velocity.sqrMagnitude > speed * speed) 
	        velocity = velocity.normalized * speed;
	        
        GetComponent<Rigidbody>().AddForce(velocity, ForceMode.VelocityChange);
        velocity = Vector3.zero;
    }
```
---
layout: page
title: Unity动画播放结束事件
---


```
animator.SetBool("IsAttack", true);
```
这是我动画播放的触发条件
我播放了一个动画，希望动画播放完成后就执行一个事件
这种事件有2种添加方法，一种就是直接修改动画，在动画中添加事件
另一种以代码判断该动画播放结束
我个人更喜欢用代码解决


这是用来判断动画播放结束的代码
```
AnimatorStateInfo stateinfo = animator.GetCurrentAnimatorStateInfo(0);
        
        if (stateinfo.IsName("attack")&& (stateinfo.normalizedTime > 1.0f))
        {
        }
```
我使用协程触发这个事件

```
StartCoroutine(WaitAttackStop());
```

```
/// <summary>
    /// 攻击动画播放完后，切换回待机状态
    /// </summary>
    /// <returns></returns>
    IEnumerator WaitAttackStop()
    {
        yield return null;
        AnimatorStateInfo stateinfo = animator.GetCurrentAnimatorStateInfo(0);
        
        if (stateinfo.IsName("attack")&& (stateinfo.normalizedTime > 1.0f))
        {
            animator.SetBool("IsAttack", false);
            if (gameobjetTag=="Enemy")
            {
               
                //待机
                StandBy();
            }
            
        }
        else
        {
	
            StartCoroutine(WaitAttackStop());
        }
    }
```


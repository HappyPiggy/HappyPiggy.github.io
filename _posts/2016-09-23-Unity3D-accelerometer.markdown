---
layout: post
title:  "在手机端,Unity如何调整重力感应？"
date:   2016-9-23 
categories: Technology
tags: accelerometer Unity3D
---

在Unity3D手机游戏开发中，有些游戏需要用到重力感应来控制主角的移动。
Unity也提供了相应的函数获得重力感应的方向。比如下面这个方式来控制重力感应。

```c#
Vector3 accleration = Input.acceleration  ;
Vector3 movement = new Vector3(accleration.x, 0.0f, accleration.y);
GetComponent<Rigidbody>().velocity = movement*speed;
```

但是这样带来了一个问题：

情况一：

当你的手机屏幕所在平面与地面平行（或稍微倾斜），此时开始游戏的话，一切正常，
你可以通过把手机倾斜到不同的方向来控制主角的移动。

情况二：

当你的手机屏幕所在平面与地面垂直时，此时开始游戏的话，你就会发现，左右倾斜时
主角还是能愉快的左右移动，但是如果你想让主角前后移动，必须以一个夸张的角度上下倾斜手机。


## 原因
可能是手机或Unity默认的重力感应器(accelerometer) 是情况一的样子。
所以当出现情况二时，用的还是情况一下的重力感应器，导致另一个方向不能完美控制。

## 解决方案
我们需要添加代码 在初始化游戏的时候  来修改重力感应器。
这样无论哪个情况都可以完美适应。

添加这两个函数
```
void ColibrateAcclerometer()
    {
        Vector3 acclerationSnapShot = Input.acceleration;
        Quaternion rotateQuaternion = Quaternion.FromToRotation(new Vector3(0.0f,0.0f,-1),acclerationSnapShot);
        calibrateQuaternion = Quaternion.Inverse(rotateQuaternion);
        Debug.Log(rotateQuaternion);
    }
Vector3 FixAccleration(Vector3 accleration)
    {
        Vector3 fixedAccleration =  calibrateQuaternion * accleration;
        return fixedAccleration;
    }
```
在Start中调用 ColibrateAcclerometer()

把需要控制主角的代码段修改为
```
Vector3 acclerationRaw = Input.acceleration  ;
Vector3 accleration = FixAccleration(acclerationRaw);
Vector3 movement = new Vector3(accleration.x, 0.0f, accleration.y);
GetComponent<Rigidbody>().velocity = movement*speed;
```

问题解决！!

   
   
    

参考[http://theantranch.com/blog/simple-iphone-calibration/](http://theantranch.com/blog/simple-iphone-calibration/)
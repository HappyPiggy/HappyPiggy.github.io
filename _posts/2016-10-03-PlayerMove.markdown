---
layout: post
title:  "实现Player移动的几种方式"
date:   2016-10-03
categories: Technology
tags: Unity3D 
---

不知道最近学校发什么神经...据说github.io结尾的访问不了..

搞我的以为我的git page瓦特掉了。。

### 控制Player移动的几种方式

一、

```c#
this.transform.Translate();
```

通过改变Player的transform属性，直接移动物体到指定位置。此方法不需要刚体组件。无惯性。

有一个和这个类似的方法，除了需要绑定刚体，其他的效果和上面的一样。这个方法通常用在`刚体失去动力学模拟的时候(即isKinametic=true)`

```c#
rigidbody.MovePosition();
```

二、

```c#
this.rigidbody.velocity=Vector3.forward;
```

通过给刚体施加速度来控制物体移动，需要绑定刚体组件，有惯性。

三、

```c#
this.rigidbody.constantForce.force=Vector3.forward;
```

通过给刚体施加力来控制物体移动，需要绑定刚体组件，有惯性且比上一种方法惯性要大。

### 让Player更圆润的移动

*  优化方法一的移动。

方法一中，比起“移动”，“瞬移”更适合用来描述它实际所起的效果。

那我们有没有什么办法让它一步一步的移动过去呢？当然是有的。假设它最终要到的位置是`targetPos`

```c#
rigidbody.MovePosition(Vector2.Lerp(transform.position,targetPos,smoothing*Time.deltaTime));
```

这里我们用了插值函数Lerp。顾名思义，从你当前的位置，和你的目标位置中插值。第三个参数是比例。

比如，我们自身位置是(0,0) 目标位置是(0,10)，第三个参数是0.1。

则第一次插值后，自身位置变为(0,1)`[解释y:10*0.1]`

第二次插值后，变为（0,1.9f）`[解释y:1+(10-1)*0.1]`。

第三次(0,2.71f)`[解释y:1.9+(10-1.9)*0.1]`......

这个过程就是自身位置不断趋近与targetPos的过程，最终到达target。Smoothing就是用来控制趋近的速度的，选取合适的值就可以达到`移动`的效果。

*  如何让方法二和方法三也圆润地滚起来？

```c#
float horizontalMove = Input.GetAxis("Horizontal");
float verticalMove = Input.GetAxis("Vertical");
Vector3 movement = new Vector3(horizontalMove,0,verticalMove);
rigidbody.AddForce(movement * speed); //或者 rigidbody.velocity = movement * speed;
```

通过GetAxis获得用户的输入，这个函数返回-1到1之间的值。比如你一直按住方向右，它会从0递增到1。这样给刚体加力或者加速度的时候也是循序渐进的加，显得更圆润。

###  总结和一些小问题

*  方法一用来控制不带惯性的物体，方法二和三用来控制带惯性的物体。

*  一般来说，物理运动应该放在FixedUpdate()里。

但是，如果你要移动的物体对时间特别敏感，我建议还是放在Update里。以下是血淋淋的教训：我在做一个人物跳的功能时，
把人物按下空格就起跳的代码放在了FixedUpdate()里，我发现有时候按空格，人物就正常起跳，但是有时候就像失灵了一样。人物根本不跳！
我个人分析是因为FixedUpdate()函数0.02s执行一次，而我按空格的时候正好它不在执行，所以人物跳不起来。当我把一切代码移入Update()后，问题解决！


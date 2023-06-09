---
title: Unity状态机
author: east
date: 2019-08-09 20:55:00 +0800
categories: [Blogging, Tutorial]
tags: [getting started]
pin: true
---

# 感同身受

假设我们在完成一个卷轴平台游戏。 现在的工作是实现玩家在游戏世界中操作的女英雄。 这就意味着她需要对玩家的输入做出响应。按B键她应该跳跃。简单实现如下：
```c#
void handleInput(Input input)
{
  if (input == PRESS_B)
  {
    yVelocity_ = JUMP_VELOCITY;
    setGraphics(IMAGE_JUMP);
  }
}
```
看到漏洞了吗？

没有东西阻止“空中跳跃”——当角色在空中时狂按B，她就会浮空。 简单的修复方法是给Heroine增加isJumping_布尔字段，追踪它跳跃的状态。然后这样做：
```c#
void Heroine::handleInput(Input input)
{
  if (input == PRESS_B)
  {
    if (!isJumping_)
    {
      isJumping_ = true;
      // 跳跃……
    }
  }
}
```

接下来，当玩家按下下方向键时，如果角色在地上，我们想要她卧倒，而松开按键时站起来：
```c++
void Heroine::handleInput(Input input)
{
  if (input == PRESS_B)
  {
    // 如果没在跳跃，就跳起来……
  }
  else if (input == PRESS_DOWN)
  {
    if (!isJumping_)
    {
      setGraphics(IMAGE_DUCK);
    }
  }
  else if (input == RELEASE_DOWN)
  {
    setGraphics(IMAGE_STAND);
  }
}
```

这次看到漏洞了吗？

通过这个代码，玩家可以：
1. 按下键卧倒。
2. 按B从卧倒状态跳起。
3. 在空中放开下键。
  
英雄跳一半贴图变成了站立时的贴图。是时候增加另一个标识了……
```c#
void Heroine::handleInput(Input input)
{
  if (input == PRESS_B)
  {
    if (!isJumping_ && !isDucking_)
    {
      // 跳跃……
    }
  }
  else if (input == PRESS_DOWN)
  {
    if (!isJumping_)
    {
      isDucking_ = true;
      setGraphics(IMAGE_DUCK);
    }
  }
  else if (input == RELEASE_DOWN)
  {
    if (isDucking_)
    {
      isDucking_ = false;
      setGraphics(IMAGE_STAND);
    }
  }
}
```
下面，如果玩家在跳跃途中按下下方向键，英雄能够做跳斩攻击就太酷了：
```c++
void Heroine::handleInput(Input input)
{
  if (input == PRESS_B)
  {
    if (!isJumping_ && !isDucking_)
    {
      // 跳跃……
    }
  }
  else if (input == PRESS_DOWN)
  {
    if (!isJumping_)
    {
      isDucking_ = true;
      setGraphics(IMAGE_DUCK);
    }
    else
    {
      isJumping_ = false;
      setGraphics(IMAGE_DIVE);
    }
  }
  else if (input == RELEASE_DOWN)
  {
    if (isDucking_)
    {
      // 站立……
    }
  }
}
```
又是检查漏洞的时间了。找到了吗？

跳跃时我们检查了字段，防止了空气跳，但是速降时没有。又是另一个字段……

我们的实现方法很明显有错。 每次我们改动代码时，就破坏些东西。 我们需要增加更多动作——行走 都还没有加入呢——但以这种做法，完成之前就会造成一堆漏洞。

# 有限状态机前来救援
在经历了上面的挫败之后，把桌子扫空，只留下纸笔，我们开始画流程图。 你给英雄每件能做的事情都画了一个盒子：站立，跳跃，俯卧，跳斩。 当角色在能响应按键的状态时，你从那个盒子画出一个箭头，标记上按键，然后连接到她变到的状态。
祝贺，你刚刚建好了一个有限状态机。 它来自计算机科学的分支自动理论，那里有很多著名的数据结构，包括著名的图灵机。 FSMs是其中最简单的成员。

要点是：

1. 你拥有状态机所有可能状态的集合。 在我们的例子中，是站立，跳跃，俯卧和速降。
  
2. 状态机同时只能在一个状态。 英雄不可能同时处于跳跃和站立状态。事实上，防止这点是使用FSM的理由之一。

3. 一连串的输入或事件被发送给状态机。 在我们的例子中，就是按键按下和松开。

4. 每个状态都有一系列的转移，每个转移与输入和另一状态相关。 当输入进来，如果它与当前状态的某个转移相匹配，机器转换为所指的状态。

举个例子，在站立状态时，按下下方向键转换为俯卧状态。 在跳跃时按下下方向键转换为速降。 如果输入在当前状态没有定义转移，输入就被忽视。

这就是核心部分的全部了：状态，输入，和转移。 你可以用一张流程图把它画出来。不幸的是，编译器不认识流程图， 所以我们如何实现一个？ GoF的状态模式是一个方法——我们会谈到的——但先从简单的开始。

# 枚举和分支
Heroine类的问题在于它不合法地捆绑了一堆布尔量： isJumping_和isDucking_不会同时为真。 但有些标识同时只能有一个是true，这提示你真正需要的其实是enum（枚举）。

在这个例子中的enum就是FSM的状态的集合，所以让我们这样定义它：
```c++
enum State
{
  STATE_STANDING,
  STATE_JUMPING,
  STATE_DUCKING,
  STATE_DIVING
};
```
不需要一堆标识，Heroine只有一个state_状态。 这里我们同时改变了分支顺序。在前面的代码中，我们先判断输入，然后 判断状态。 这让处理某个按键的代码集中到了一处，但处理某个状态的代码分散到了各处。 我们想让处理状态的代码聚在一起，所以先对状态做分支。这样的话：

```c++
void Heroine::handleInput(Input input)
{
  switch (state_)
  {
    case STATE_STANDING:
      if (input == PRESS_B)
      {
        state_ = STATE_JUMPING;
        yVelocity_ = JUMP_VELOCITY;
        setGraphics(IMAGE_JUMP);
      }
      else if (input == PRESS_DOWN)
      {
        state_ = STATE_DUCKING;
        setGraphics(IMAGE_DUCK);
      }
      break;

    case STATE_JUMPING:
      if (input == PRESS_DOWN)
      {
        state_ = STATE_DIVING;
        setGraphics(IMAGE_DIVE);
      }
      break;

    case STATE_DUCKING:
      if (input == RELEASE_DOWN)
      {
        state_ = STATE_STANDING;
        setGraphics(IMAGE_STAND);
      }
      break;
  }
}
```
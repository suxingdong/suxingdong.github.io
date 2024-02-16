---
title: Unity Animancer
author: East.Su
date: 2023-02-14 18:32:00 -0500
categories: [Unity, Animatior]
tags: [Animancer]
---

[官方文档] (https://kybernetik.com.au/animancer/docs/introduction/)

 # 快速开始
 ```c#
    using Animancer;
    using UnityEngine;
    public sealed class PlayAnimationOnEnable : MonoBehaviour
    {
        [SerializeField] private AnimancerComponent _Animancer;
        [SerializeField] private AnimationClip _Animation;

        private void OnEnable()
        {
            _Animancer.Play(_Animation);
        }
    }
```

# 简单移动
```c#
    using Animancer;
    using UnityEngine;

    public sealed class BasicMovementAnimations : MonoBehaviour
    {
        [SerializeField] private AnimancerComponent _Animancer;
        [SerializeField] private AnimationClip _Idle;
        [SerializeField] private AnimationClip _Move;

        private void Update()
        {
            float forward = ExampleInput.WASD.y;
            if (forward > 0)
            {
                _Animancer.Play(_Move);
            }
            else
            {
                _Animancer.Play(_Idle);
            }
        }
    }
```
# Transitions
```c#
[SerializeField]
private AnimancerComponent _Animancer;

[SerializeField]
private ClipTransition _Idle;

[SerializeField]
private ClipTransition _Action;

private void OnEnable()
{
    _Action.Events.OnEnd = OnActionEnd;

    _Animancer.Play(_Idle);
}

	
private void OnActionEnd()
{
    _Animancer.Play(_Idle);
}

private void Update()
{
    if (ExampleInput.LeftMouseUp)
    {
        _Animancer.Play(_Action);
    }
}
```

# Basic Character
```c#
sing Animancer;
using UnityEngine;

public sealed class BasicCharacterAnimations : MonoBehaviour
{
    [SerializeField] private AnimancerComponent _Animancer;
    [SerializeField] private ClipTransition _Idle;
    [SerializeField] private ClipTransition _Move;
    [SerializeField] private ClipTransition _Action;

    private enum State
    {
        NotActing,
        Acting,
    }

    private State _CurrentState;

    private void Awake()
    {
        _Action.Events.OnEnd = OnActionEnd;
    }
    
    private void OnActionEnd()
    {
        _CurrentState = State.NotActing;
        UpdateMovement();
    }

    private void Update()
    {
        switch (_CurrentState)
        {
            case State.NotActing:
                UpdateMovement();
                UpdateAction();
                break;

            case State.Acting:
                UpdateAction();
                break;
        }
    }

    private void UpdateMovement()
    {
        _CurrentState = State.NotActing;

        float forward = ExampleInput.WASD.y;
        if (forward > 0)
        {
            _Animancer.Play(_Move);
        }
        else
        {
            _Animancer.Play(_Idle);
        }
    }

    private void UpdateAction()
    {
        if (ExampleInput.LeftMouseUp)
        {
            _CurrentState = State.Acting;
            _Animancer.Play(_Action);
        }
    }
}
```


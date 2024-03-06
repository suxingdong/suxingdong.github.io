---
title: Unity PhotonEngine-Quantum
author: East.Su
date: 2024-03-06 17:55:00 +0800
categories: [Unity, Act]
tags: [PhotonEngine]
---

 [官网](https://dashboard.photonengine.com/zh-cn)
 
 <iframe width="640" height="390" src="https://www.youtube.com/embed/oVqbPnG70qc" title="Hello Quantum 2.0" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

# <font size=40>Quantum</font>
+ 广泛使用于RTS，MOBA，格斗动作，体育运动类游戏。
+ 价格：
  1. Public Cloud(公共云)
    a. 免费20个ccu。    
	b. $95 提供12个月100个ccu预估支持40,000个用户。   
	c. $1,250 提供12个月100个ccu预估支持40,000个用户。  
	d. $ 2,500 提供12个月1000个ccu预估支持400,000个用户。  
	f. $ 5,000 提供12个月2000个ccu预估支持800,000个用户。  

  2. Premium Cloud(高性能云)   
	  $1,000  提供1个月2000个ccu。
  3. Self-host(本地服务器)
	需要是Gaming Circle，获取Free Development License


### ECS
Quantum是采用ECS框架，ECS独立于UnityEngine被放在quantum_code项目中,通过dsl生成component，这些组件通过unity中EntityPrototype挂在在角色身上，系统(继承SystemMainThread)通过SystemSetup注册到Core.cs, Quantum引擎回回调这些System函数。

#### 通过DSL生成数据类代码
	1. 注册账号 生成appid
	2. 下载测试案例 (https://dashboard.photonengine.com/zh-cn/download/quantum/quantum-100-2.1.4.zip)
	3. 解压文件quantum-100-2.1.4.zip，将quantum-100-2.1.4\tools\codeintegration_unity内容全部复制到quantum-100-2.1.4\quantum_code\quantum.code
	4. 在quantum_unity/Packages/manifest.json中添加本地配置包配置 "com.exitgames.photonquantumcode": "file:../../quantum_code/quantum.code"
	5. 删除 quantum_unity/Assets/Photon/Quantum/Assemblies/quantum.code.dll
	6. quantum-100-2.1.4\quantum_code\quantum.code\quantum.code.csproj 添加下面代码
		<ItemGroup>
			<None Include="Foo\bar.qtn" />
			<None Include="Oof\rab.qtn" />
		</ItemGroup>
	7. 创建qtn文件
   
```c#
	component Action
		{
			FP Cooldown;
			FP Power;
		}
```
	8. cmd运行 D:\quantum-100-2.1.4/tools/codegen/quantum.codegen.host.exe D:\quantum-100-2.1.4\quantum_code\quantum.code 
	9. 打开quantum_code.sln 生成dll文件，开发阶段可以不用生成
![alt text](/assets/image-8.png)
	1. 使用Unity打开quantum_unity 生qtn代码
![alt text](/assets/image-6.png)

#### 组件 (Components)
+ 自定义组件 通过dsl生成  
```c#
	component Action  
	{ 
		FP Cooldown;
		FP Power;
	}
```	
	生成组件名：EntityComponentAction
+ 系统组件 无状态物理引擎 PhysicsCollider、PhysicsBody、PhysicsCallbacks、PhysicsJoints （2D/3D） 
+ 系统组件 基于导航网格的路径查找和移动 PathFinderAgent、SteeringAgent、AvoidanceAgent、AvoidanceObstacle：。
 

#### 游戏逻辑 Systems 
+ 继承SystemMainThread
+ 继承SystemMainThreadFilter<T> ,可以关注需要的内容
+ System需要通过SystemSetup:CreateSystems需要将数据放入Core.cs   
+ Systems通过Update从Frame中获得需要的数据：f.Unsafe.TryGetPointer  
  
```c#
public unsafe class MySystem : SystemMainThread
  {
    public override void Update(Frame f)
    {
    }
  }

public unsafe class MovementSystem : SystemMainThreadFilter<MovementSystem.Filter>
{
    public struct Filter
    {
      public EntityRef Entity;
      public Transform2D* Transform;
      public PlayerCharacter* PlayerCharacter;
      public KCC* Kcc;
    }
	
	public override void Update(Frame frame, ref Filter filter)
	{
		//frame.Unsafe.GetPointer<PlayerCharacter>(entity);
	}
}
```

#### Frame
	frame是系统(继承SystemMainThread)通过Quantum内部回调Update得到。
	继承关系：Frame:FrameBase:DeterministicFrame
	使用Frame.User 扩展Frame数据
	System从Frame中获得需要的数据：f.Unsafe.TryGetPointer

#### 事件和回调 （Events & Callbacks）
```c#
	//dsl生成myevnt类
	event MyEvent {
		int Foo;
	}
	//事件触发 必须是系统（MySystem）
	public override void Update(Frame f) {
	f.Events.MyEvent(2022);
	}
	//事件注册
	QuantumEvent.Subscribe<EventMyEvent>(listener: this, handler: OnEventPlayerHit);

	private void OnEventPlayerHit(EventMyEvent e){
	Debug.Log($"MyEvent {e.Foo}");
	}
```

#### 定点数 Events & Callbacks
+ 采用的是Q48.16（48位整数，16位小数）
+ 使用FPAnimationCurve 创建动画曲线，或者AnimationCurve 烘培到FPAnimationCurve

### QuantumLoadBalancingClient 使用(点击开始按钮创建QuantumLoadBalancingClient)
	1. var Client = new QuantumLoadBalancingClient(PhotonServerSettings.Instance.AppSettings.Protocol);
	2. Client.ConnectUsingSettings(appSettings, Username)
	3. Client.OpJoinRandomOrCreateRoom(joinRandomParams, _enterRoomParams)
	4. QuantumRunner.StartGame(clientId, param);

### EntityViewUpdater （渲染部分）
	1. 注册消息 QuantumCallback.Subscribe
	2. 同步数据 SyncViews （是否创建player BindMapEntityIfNeeded->CreateView->CreateEntityViewInstance->GameObject.Instantiate）
	

###	EntityPrototype 
	这是一个实体原型，虽然继承了MonoBehaviour，但是没有使用任何API，直接用于挂在组件而用的，实际数据序列化到EntityPrototypeAsset。	
	需要加载EntityPrototypeAsset数据，实例化组件
		IResourceLoader.LoadResource(AssetResource resource)
		FinishLoading
		EntityPrototypeAsset:PrepareAsset()->behaviourBuffer

### EntityView 渲染部分
	Non Verified：可在预测帧中创建视图游戏对象，如子弹
	Verified ：只能在已验证帧中创建，如玩家角色	

### QuantumCallbacks 相当于MonoBehaviour，回调数据来说photon网络调用	
+ Quantum.Core Quantum 内部调用的一些接口
```c#
   public enum CallbackId {
    PollInput,
    GameStarted,//游戏开始时调用
    GameResynced,//游戏快照重新同步时调用的回调
    GameDestroyed,//游戏销毁时调用
    UpdateView,//回调保证在每个渲染帧被调用。
    SimulateFinished,//逻辑帧完成回调
    EventCanceled, //当由于回滚,错过预测而在已验证帧中取消预测帧中引发的事件时调用的回调
    EventConfirmed,//当经过验证的帧确认事件时调用回调
    ChecksumError,//校验预测错误时调用回调
    ChecksumErrorFrameDump,//由于校验预测错误而转储帧时调用的回调
    InputConfirmed,//本地输入确认后回调
    ChecksumComputed,
    PluginDisconnect,
    UserCallbackIdStart,
  }
```
+ QuantumCallbacks 函数  
		OnGameStart:  
		OnGameResync: 
		OnGameDestroyed: 
		OnUpdateView:  
		OnSimulateFinished:  
		OnUnitySceneLoadBegin:  
		OnUnitySceneLoadDone:  
		OnUnitySceneUnloadBegin:  
		OnUnitySceneUnloadDone:  
		OnChecksumError:  
+ 注册数据Frame (quantum_code里面)  
		_ISignalOnPlayerDataSet = BuildSignalsArray<ISignalOnPlayerDataSet>();  
		_ISignalOnCollision2DSystems      = BuildSignalsArray<ISignalOnCollision2D>();  
		_ISignalOnCollisionExit2DSystems  = BuildSignalsArray<ISignalOnCollisionExit2D>();  
+ 发送数据  
		发生数据：game.SendPlayerData(lp, new Quantum.RuntimePlayer { PlayerName = PlayerName, SelectedCharacter = CharacterPrototype});

+ 接受数据   
		Core:OnSimulate()
		Core:UpdatePlayerData()-> ISignalOnPlayerDataSet:OnPlayerDataSet() (quantum_code里面)	
		派发到 PlayerConnectedSystem 里面
+ QuantumCallbackHandler_LegacyQuantumCallback   
   		QuantumCallback:QuantumCallback:() //内部调用
		IsDefaultHandlerEnabled()
		QuantumCallbackHandler_LegacyQuantumCallback.Initialize()
		QuantumCallback.SubscribeManual //手动订阅消息（CallbackChecksumError，CallbackGameDestroyed，CallbackGameStarted，CallbackGameResynced ...）

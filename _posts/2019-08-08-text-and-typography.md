---
title: AssetBundle 资源打包
author: east.su
date: 2019-08-08 11:33:00 +0800
categories: [Blogging, Unity, C#]
tags: [typography]
math: true
mermaid: true
image:
  path: /commons/devices-mockup.png
  lqip: 
  alt: Responsive rendering of Chirpy theme on multiple devices.
---
# AssetBundle介绍
AssetBundle是Unity提供的一种用于存储资源的压缩格式的存档文件，彼此之间的依赖关系，通常内置使用LZMA和LZ4压缩格式，包括内容序列化文件，模型、纹理、预制件、音频剪辑，甚至整个场景

# AssetBundles 工作流
1. 创建AssetBundle文件。
2. 将打包好的AssetBundle文件拷贝到StreamingAssets，或者上传到服务器
3. 加载AssetBundle文件(通过本地数据，或从服务下载)
4. 卸载AssetBundle


## 创建AssetBundles
1. 创建Assetbundle文件对象

- 第一种：使用 Inspector 设置打包对象
![image](./assetbundle/asstebundle1.jpg)

- 第二种：通常需要制定打包策略，使用AssetImporter整理资源对其进行打包

```C#
[System.Serializable]
public class PackageItem
{
    public enum PackType {File,DirFiles,Dir}
    public PackType type;
    public string path;
    public string searchPattern = "*.*";
    public SearchOption searchOption;
}

[System.Serializable]
public class PackageConfig : ScriptableObject {

    private static PackageConfig _instance;
    public  static PackageConfig instance
    {
        get
        {
            _instance = AssetDatabase.LoadAssetAtPath<PackageConfig>("Assets/Tools/Editor/PackConf.asset");
            if(_instance==null)
            {
                _instance = ScriptableObject.CreateInstance<PackageConfig>();
                AssetDatabase.CreateAsset(_instance, "Assets/Tools/Editor/PackConf.asset");
            }

            return _instance;
        }
    }

    //版本号
    [SerializeField]
    private string _version = "1.0";
    public static string version { get { return instance._version; } }

    //游戏名称
    [SerializeField]
    private string _appName = "Test";
    public static string appName { get { return instance._appName; } }


    //游戏发布的目录
    [SerializeField]
    private string _publishDir = "publish/";
    public static string publishDir { get { return instance._publishDir; } }

    //游戏web发布目录
    [SerializeField]
    private string _webPublishUrl = "http://192.168.1.27/publish/";
    public static string webPulishUrl { get { return instance._webPublishUrl; } }

    // 是否使用jit
    [SerializeField]
    private bool _luaJit = true;
    public  bool luaJit { get { return instance._luaJit; } }

    // debugBuild
    [SerializeField]
    private bool _debugBuild = true;
    public static bool debugBuild { get { return instance._debugBuild; } }

    // 自动开启
    [SerializeField]
    private bool _autoRunPlayer = false;
    public static bool autoRunPlayer { get { return instance._autoRunPlayer; } }

    [SerializeField]
    private List<PackageItem> _packItems = new List<PackageItem>();
    public  List<PackageItem> packItems { get { return instance._packItems; } }

    public static BuildOptions buildOptions
    {
        get
        {
            if(debugBuild)
            {
                BuildOptions op = BuildOptions.AllowDebugging | BuildOptions.ConnectWithProfiler | BuildOptions.Development;
                if(autoRunPlayer)
                {
                    return op | BuildOptions.AutoRunPlayer;
                }
                return op;
            }
            else
            {
                return BuildOptions.None;
            }
        }
    }
   
}

  // 各种配置文件
[MenuItem("BuildTool/依赖打包方式/发布设定")]
public static void open()
{
    Selection.activeObject = PackageConfig.instance;
}

```



单个文件打包，通过AssetImporter.GetAtPath获得AssetImporter，设置assetBundleName。

```C#
    //文件打包
    public bool PackFile(string res)
    {
        AssetImporter importer = AssetImporter.GetAtPath(res);
        if (importer == null)
        {
            Debug.LogError("Path not Exist!" + res);
            return false;
        }
        importer.assetBundleName = res + abExtens;
        return true;
    }
```

获得所有的AB包：

```C#
string[] assetBundleNames = AssetDatabase.GetAllAssetBundleNames();
```

调用打包api：BuildPipeline.BuildAssetBundles

```C#
using UnityEditor;
using UnityEngine;
using System.IO;

public class CreateAssetBundles
{
	[MenuItem("Assets/打包全局对象")]
	static void BuildAllAssetBundles()
	{
  	string assetBundleDirectory = "Assets/StreamingAssets"; // 包的输出路径
  	if (!Directory.Exists(Application.streamingAssetsPath)) 
  	{
    	Directory.CreateDirectory(assetBundleDirectory);// 若路径不存在，则创建
  	}
    // BuildPipeline：允许您以编程方式构建可从 Web 加载的播放器或 AssetBundle。
    // BuildAssetBundles()：打包，Build 出来的包是有平台限制的
  	BuildPipeline.BuildAssetBundles(assetBundleDirectory, BuildAssetBundleOptions.None, EditorUserBuildSettings.activeBuildTarget);
	}

    [MenuItem("Assets/打包指定对象")]
    static void BuildMapABs()
    {
        AssetBundleBuild[] buildMap = new AssetBundleBuild[2];

        buildMap[0].assetBundleName = "enemybundle";
        string[] enemyAssets = new string[2];
        enemyAssets[0] = "Assets/Textures/char_enemy_alienShip.jpg";
        enemyAssets[1] = "Assets/Textures/char_enemy_alienShip-damaged.jpg";
        buildMap[0].assetNames = enemyAssets;

        buildMap[1].assetBundleName = "herobundle";
        string[] heroAssets = new string[1];
        heroAssets[0] = "char_hero_beanMan";
        buildMap[1].assetNames = heroAssets;

        BuildPipeline.BuildAssetBundles("Assets/StreamingAssets", buildMap, BuildAssetBundleOptions.None, BuildTarget.activeBuildTarget);
    }
}


```


## 加载资源

1. 加载依赖文件

```C#
private void Start() { 
   
     AssetBundle.LoadFromFile("Assets/AssetBundles/share.u3d"); // 加载依赖包
     AssetBundle ab = AssetBundle.LoadFromFile("Assets/AssetBundles/cube.u3d");

     var wallPre = ab.LoadAsset<GameObject>("cube");
	 Instantiate(wallPre); // 实例化物体
 }
```
2. 加载本地的 AB 包

```C#
using System.IO;
using UnityEngine;

public class LoadFromFileExample : MonoBehaviour { 
   

    private void Start() { 
   
        AssetBundle ab = AssetBundle.LoadFromFile("Assets/AssetBundles/cube.bundle");
        if (ab == null) { 
   
            Debug.Log("Failed to load AssetBundle!");
            return;
        }

        // 加载包里的指定物体
        var wallPre = ab.LoadAsset<GameObject>("cube");
        Instantiate(wallPre); // 实例化物体
    }

    // 加载包里的所有物体
    private void LoadAllAssets(AssetBundle ab) { 
   
        Object[] objects = ab.LoadAllAssets();
        foreach (var o in objects) { 
   
            Instantiate(o);
        }
    }

    //加载从服务器下载资源包
    IEnumerator InstantiateObject()
    { 
   
        string uri = "file:///" + Application.dataPath + "/AssetBundles/" + assetBundleName;
        // GetAssetBundle(string, int)：获取 AssetBundle 的位置以及要下载的捆绑包的版本。
        UnityEngine.Networking.UnityWebRequest request = UnityEngine.Networking.UnityWebRequest.GetAssetBundle(uri, 0);
        yield return request.Send();
        AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(request);
        GameObject cube = bundle.LoadAsset<GameObject>("Cube");
        GameObject sprite = bundle.LoadAsset<GameObject>("Sprite");
        Instantiate(cube);
        Instantiate(sprite);
    }
}
```
 

##  卸载AssetBundle
 - AssetBundle.Unload(false) 会把 Bundle 卸载，但是已经从 Bundle 里加载出来的 资源 是不会被卸载的。
 - AssetBundle.Unload(true) 不但会卸载 Bundle，也会卸载已经从 Bundle 里加载出来的 所有资源，哪怕这些 资源 还被引用着。
 -  Resources.UnloadUnusedAssets 接口帮助我们销毁没有任何引用的 野资源，不过这个函数会扫描全部对象，开销较大，一般只在 切场景 时调用。

# AssetBundle 浏览工具
https://github.com/Unity-Technologies/AssetBundles-Browser.git

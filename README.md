# SingletonKit

一套单例工具，易上手且功能强大。

由 QFramework 团队官方维护的独立工具包（不依赖 QFramework）。

## 环境要求

* Unity 2018.4LTS

## 安装

* PackageManager
    * add from package git url：https://github.com/liangxiegame/SingletonKit.git 
    * 或者国内镜像仓库：https://gitee.com/liangxiegame/SingletonKit.git
* 或者直接复制[此代码](SingletonKit.cs)到自己项目中的任意脚本中

## 使用指南

**1. Singleton**

``` csharp
namespace QFramework.Example
{
	using UnityEngine;

    internal class Class2Singleton : Singleton<Class2Singleton>
	{
		private static int mIndex = 0;

		private Class2Singleton() {}

		public override void OnSingletonInit()
		{
			mIndex++;
		}

		public void Log(string content)
		{
			Debug.Log("Class2Singleton" + mIndex + ":" + content);
		}
	}
	
	public class Singleton : MonoBehaviour
	{
		private void Start()
		{
			Class2Singleton.Instance.Log("Hello World!");
			
			// delete instance
			Class2Singleton.Instance.Dispose();
			
			// a differente instance
			Class2Singleton.Instance.Log("Hello World!");
		}
	}
}
```

运行结果

控制台:
```
Class2Singleton1:Hello World!
Class2Singleton2:Hello World!
```

**2.MonoSingleton**

``` csharp
namespace QFramework.Example
{
	using System.Collections;
	using UnityEngine;

	internal class Class2MonoSingleton : MonoSingleton<Class2MonoSingleton>
	{
		public override void OnSingletonInit()
		{
			Debug.Log(name + ":" + "OnSingletonInit");
		}

		private void Awake()
		{
			Debug.Log(name + ":" + "Awake");
		}

		private void Start()
		{
			Debug.Log(name + ":" + "Start");
		}

		protected override void OnDestroy()
		{
			base.OnDestroy();
			
			Debug.Log(name + ":" + "OnDestroy");
		}
	}

	public class MonoSingletonExample : MonoBehaviour
	{
		private IEnumerator Start()
		{
			var instance = Class2MonoSingleton.Instance;

			yield return new WaitForSeconds(3.0f);
			
			instance.Dispose();
		}
	}
}
```

运行结果:

场景结构(Hierarchy)
```
Scene
    MonoSingleton
DontDestroyOnLoad（3 秒钟后消失）
    Class2MonoSingleton
```

控制台
```
Class2MonoSingleton:Awake
Class2MonoSingleton:OnSingletonInit
Class2MonoSingleton:Start
3 秒中后
Class2MonoSingleton:OnDestroy
```

**3.SingletonProperty**
``` csharp
namespace QFramework.Example
{
	using UnityEngine;

	internal class Class2SignetonProperty : ISingleton
	{
		public static Class2SignetonProperty Instance
		{
			get { return SingletonProperty<Class2SignetonProperty>.Instance; }
		}

		private Class2SignetonProperty() {}
		
		private static int mIndex = 0;

		public void OnSingletonInit()
		{
			mIndex++;
		}

		public void Dispose()
		{
			SingletonProperty<Class2SignetonProperty>.Dispose();
		}
		
		public void Log(string content)
		{
			Debug.Log("Class2SingletonProperty" + mIndex + ":" + content);
		}
	}
		
	public class SingletonProperty : MonoBehaviour
	{
		// Use this for initialization
		void Start () 
		{
			Class2SignetonProperty.Instance.Log("Hello World!");	
			
			// delete current instance
			Class2SignetonProperty.Instance.Dispose();
			
			// new instance
			Class2SignetonProperty.Instance.Log("Hello World!");
		}
	}
}
```

结果:

控制台
```
Class2SingletonProperty1:Hello World!
Class2SingletonProperty2:Hello World!
```

**4.MonoSingletonProperty**

``` csharp
namespace QFramework.Example
{
	using System.Collections;
	using UnityEngine;

	internal class Class2MonoSingletonProperty : MonoBehaviour,ISingleton
	{
		public static Class2MonoSingletonProperty Instance
		{
			get { return MonoSingletonProperty<Class2MonoSingletonProperty>.Instance; }
		}
		
		public void Dispose()
		{
			MonoSingletonProperty<Class2MonoSingletonProperty>.Dispose();
		}
		
		public void OnSingletonInit()
		{
			Debug.Log(name + ":" + "OnSingletonInit");
		}

		private void Awake()
		{
			Debug.Log(name + ":" + "Awake");
		}

		private void Start()
		{
			Debug.Log(name + ":" + "Start");
		}

		protected void OnDestroy()
		{
			Debug.Log(name + ":" + "OnDestroy");
		}
	}

	public class MonoSingletonProperty : MonoBehaviour
	{
		private IEnumerator Start()
		{
			var instance = Class2MonoSingletonProperty.Instance;

			yield return new WaitForSeconds(3.0f);
			
			instance.Dispose();
		}
	}
}
```

运行结果

场景结构(Hierarchy)
```
Scene
    MonoSingletonProperty
DontDestroyOnLoad（3 秒钟后消失）
    Class2MonoSingletonProperty
```

控制台
```
Class2MonoSingletonProperty:Awake
Class2MonoSingletonProperty:OnSingletonInit
Class2MonoSingletonProperty:Start
3 秒中后
Class2MonoSingletonProperty:OnDestroy
```

**5.MonoSingletonPath**
``` csharp
namespace QFramework.Example
{
	using UnityEngine;

	[QFramework.MonoSingletonPath("[Example]/MonoSingletonPath")]
    internal class ClassUseMonoSingletonPath : MonoSingleton<ClassUseMonoSingletonPath>
	{
		
	}
	
	public class MonoSingletonPath : MonoBehaviour
	{
		private void Start()
		{
			var intance = ClassUseMonoSingletonPath.Instance;
		}
	}
}
```

运行结果

场景结构(Hierarchy)
```
Scene
    MonoSingletonPath
DontDestroyOnLoad
    [Example]
        MonoSingletonPath
```

**6.PersistentMonoSingleton**
``` csharp
using System.Collections;
using UnityEngine;

namespace QFramework.Example
{
	public class PersistentMonoSingletonExample : MonoBehaviour
	{
		// Use this for initialization
		IEnumerator Start()
		{
			// 创建一个单例
			var instance = GameManager.Instance;

			// 强制创建一个实例
			new GameObject().AddComponent<GameManager>();

			// 等一帧，等待第二个 GameManager 把自己删除
			yield return new WaitForEndOfFrame();
			
			// 结果为 1 
			Debug.Log(FindObjectsOfType<GameManager>().Length);
			
			// 保留最先创建的实例
			Debug.Log(instance == FindObjectOfType<GameManager>());
		}


		public class GameManager : PersistentMonoSingleton<GameManager>
		{

		}
	}

}
```

结果

控制台
``` 
1
True
```

**7.ReplaceableMonoSingleton**

``` csharp
using System.Collections;
using UnityEngine;

namespace QFramework.Example
{
	public class ReplaceableMonoSingletonExample : MonoBehaviour
	{
		// Use this for initialization
		IEnumerator Start()
		{
			// 创建一个单例
			var instance = GameManager.Instance;

			// 强制创建一个实例
			new GameObject().AddComponent<GameManager>();

			// 等一帧，等待第二个 GameManager 把自己删除
			yield return new WaitForEndOfFrame();
			
			// 结果为 1 
			Debug.Log(FindObjectsOfType<GameManager>().Length);
			
			// 最先创建的实例已经被删除
			Debug.Log(instance != FindObjectOfType<GameManager>());
		}


		public class GameManager : ReplaceableMonoSingleton<GameManager>
		{

		}
	}
}
```

结果

控制台：
```
1
True
```



## 更多

* QFramework 地址: https://github.com/liangxiegame/qframework
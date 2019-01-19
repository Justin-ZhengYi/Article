### iOS基础之Bundle详解   
在iOS工程中，如果我们使用xib来编写视图的话，会经常用到Bundle.main.loadNibNamed的方法来加载视图。那么Bundle到底是什么呢？除了main bundle还有其他bundle？    
这篇文章是对Bundle的讲解，主要说明Bundle的定义和特点，Bundle的内部结构以及Bundle的基本使用。
#### 1 Bundle的定义和特点      
Bundle是一个含有可执行的代码及代码所需资源，以特定标准的层次结构组合起来的文件夹。这里的可执行是指编译过后可直接运行的代码程序。一个典型的例子就是iOS程序打包后的ipa，我们解压ipa后会得到一个payload文件夹，进入文件夹之后会发现一个和ipa名称相同的.app文件加。这个.app文件夹就是一个Bundle。  
系统如何识别Bundle呢？一般而言一个文件夹如果带着.app，.bundle，.framework，.plugin，.kext等等特定后缀，那么系统就认为是Bundle。如果使用Xcode创建项目的话，Xcode会提供相应的模板来生成正确的Bundle类型。  
从上面的后缀我们也可以看出Bundle主要分为：  

* Appliction。应用程序，包含代码和资源。iOS和macOS的app就是这种。    
* Frameworks。框架，包含动态共享库和相应资源。我们常用的系统库和第三方库都属于这种。  
* Plug-Ins。插件，macOS很多的系统功能支持插件，一种动态加载代码模块的方式。     

使用Bundle可以很方便的管理程序的文件内容，进行本地化设置、程序移动和运行等等。   

#### 2 Bundle的内部结构    
##### 2.1 iOS Application Bundle的结构  
Application类型的bundle是很常见的，一般里面会包含一下几中类型的文件：  

* Info.plist文件。每个程序中必须有这个文件，因为它包含了程序运行的配置信息，是系统运行程序的依据。  
* 可执行代码文件。这是程序的主体，包含了程序的进入点和链接的静态代码。  
* 资源文件。程序运行过程中需要的资源，比如图片，音频，视频，多语言的字符串文件，nib文件，数据文件，配置文件等等。这里面的大部分文件都可以根据语言、地区、设备通过特定的结构或命名方式加以区分，程序会自动识别加载。  
* 其他支持文件。Mac app可以嵌入高层资源，比如私有库，插件，文件末班，自定义数据资源等等。iOS app可以包含自定义数据资源，但是不能包含私有库和插件。    

如果我们对于ipa中的.app文件夹右键显示包内容，我们可以看到app bundle的整体结构如下：  

	MyApp  
	  MyApp  //代码编译链接后的可执行文件  
	  Info.plist //程序信息
	  Assets.car   //Assets中的资源对应的文件。    
	  MyAppIcon.png  //所有的icon图片   
	  LaunchImage.png  //所有启动图片 
	  nibs  //工程中的xib，storyboard生成的nib文件 
	  customResource  //工程目录下或者更里层文件夹下的图片，音、视频，数据文件等自定义资源  
	  customBundle.bundle  //工程中使用的其他第三方或资源的bundle  
	  en.lproj  //语言区分的文件夹，可以使字符串、图片，nib文件等
	    MyString  
	    MyImage
	  zh-Hans.lproj  
	    MyString  
	    MyImage  
	  PkgInfo  //系统识别信息  
	  _CodeSignature  //签名信息文件夹，里面的CodeResources包含对bundle中的所有资源文件的签名信息  
	  embedded.mobileprovision //打包的配置文件信息  
	  
	  

Assets.car无法直接打开，可以使用工具 [cartool](https://github.com/steventroughtonsmith/cartool) 列出包含的所有文件名称。PkgInfo、签名信息和嵌入配置文件都是打包后生成的，是系统验证app的依据。  
  
##### 2.2 Framework Bundle结构  
Framework Bundle和app bundle的最大不同在于框架库有版本控制，因此必须包含版本列表和当前版本信息。  
Framework的bundle结构如下：  

	MyFramework.framework/
		MyFramework -> Versions/Current/MyFramework  
		Resources   -> Versions/Current/Resources  
		Versions/
			A/
				MyFramework
				Headers/
					MyHeader.h
				Resources/ 
					Eglish.lproj/
						InfoPlist.strings 
					Info.plist
			Current -> A

上面的根目录下的MyFramework和Resources，以及Versions下面的current 都是文件引用，表明引用文件的位置。真正的代码，资源，header等内容都放在Versions文件夹下面的具体版本文件夹中，Current就指向当前版本的引用。    

关于多语言本地化文件，是按照*language_region.lproj*的格式命名的，比如en_GB表明英语_英国，zh_Hans表示中文_中国大陆。  
  
#### 3 Bundle的基本使用    
使用bundle主要都是围绕着定位资源路径而来的。包括：获取bundle及其信息（不管是main bundle还是其他bundle），获取资源路径（根据资源名称和类型定位），使用系统API直接获取资源（比如图片、音频、本地化字符串、context help，nibs，Info.plist内容等等），具体使用可以参考 [Class Bundle](https://developer.apple.com/documentation/foundation/bundle)文档，也可以参考我的[Demo](https://github.com/Justin-ZhengYi/BundelPathDemo)。    

在磁盘上查找资源时，Bundle对象遵循特定的搜索模式。首先返回全局资源（即不在特定于语言的.lproj目录中的资源），然后返回特定于区域和语言的资源。此搜索模式表示捆绑包按以下顺序查找资源：

* 全局（非本地化）资源
* 特定于区域的本地化资源（基于用户的区域首选项）
* 特定于语言的本地化资源（基于用户的语言首选项）
* 开发语言资源（由bundle的Info.plist文件中的CFBundleDevelopmentRegion键指定）

由于全局资源优先于特定于语言的资源，因此您不应在应用程序中同时包含给定资源的全局和本地化版本。存在资源的全局版本时，永远不会返回特定于语言的版本。这种优先权的原因是性能。如果首先搜索本地化资源，则bundle对象可能会浪费时间在返回全局资源之前搜索不存在的本地化资源。

在查找资源文件时，bundle对象在确定要返回的文件时会自动考虑许多标准文件名修饰符。可以为特定设备（~iphone，~ipad）或特定屏幕分辨率（@2x，@ 3x）标记资源。指定所需资源的名称时，请不要包含这些修饰符。 bundle对象选择最适合底层设备的文件。   

 


参考资料：  
1.[Bundle Programming Guide](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/AccessingaBundlesContents/AccessingaBundlesContents.html#//apple_ref/doc/uid/10000123i-CH104-SW6)  
2.[Class Bundle Reference](https://developer.apple.com/documentation/foundation/bundle)  




  

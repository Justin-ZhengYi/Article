### UML类图详解  

> 我们在阅读开源项目时总是希望能比较高效的整理清楚项目中的各个类之间的关系，那么有没有相应的工具能高效、简洁的表示清楚类关系呢？UML类图就是一个可以帮我们解决此类问题的工具或者方法。   

#### 1 什么是UML
  
统一建模语言（Unified Modeling Language，缩写UML）是非专利的第三代建模和规约语言。  
UML是一种开放的方法，用于说明、可视化、构建和编写一个正在开发的、面向对象的、软件密集系统的制品的开放方法。

**UML模型和图形**

UML分为模型和图形两大类。区分UML模型和UML图是非常重要的，UML图（包括用例图、协作图、活动图、序列图、部署图、构件图、类图、状态图）是模型中信息的图表表达形式，但是UML模型独立于UML图存在。     

在UML系统开发中有三个主要的模型：  

* **功能模型**：从用户的角度展示系统的功能，包括用例图。   
* **对象模型**：采用对象，属性，操作，关联等概念展示系统的结构和基础，包括类别图、对象图。   
* **动态模型**：展现系统的内部行为。包括序列图，活动图，状态图。   

UML2.2中一共定义了14种图示。   

结构性图形（*Structure diagrams*）强调的是系统式的建模：  

*  静态图（static diagram)：包括类图、对象图、包图  
*  实现图（implementation diagram）：包括组件图、部署图  
*  剖面图
*  复合结构图  

行为式图形（*Behavior diagrams*）强调系统模型中触发的事件  

* 活动图
* 状态图
* 用例图

交互性图形（*Interaction diagrams*），属于行为图形的子集合，强调系统模型中的资料流程  

* 通信图
* 交互概述图
* 时序图
* 时间图  

#### 2 UML类图作用  

UML展现了一系列最佳工程实践，这些最佳实践在对大规模，复杂系统进行建模方面，特别是软件架构层次方面已经被验证有效。

我们这次介绍的主要是类图，为了解析项目的系统结构和架构层次，可以简洁明了的帮助我们理解项目中类之间的关系。     

类图的作用：   
（1）：在软件工程中，类图是一种静态的结构图，描述了系统的类的集合，类的属性和类之间的关系，可以简化了人们对系统的理解；   
（2）：类图是系统分析和设计阶段的重要产物，是系统编码和测试的重要模型。  
​
#### 3 类图格式    
在UML类图中，类使用包含类名、属性(field) 和方法(method) 且带有分割线的矩形来表示，

举个栗子。一个Animal类，它包含name,age和sex这3个属性，以及name相关方法。  
	  

	class Animal: NSObject {
    
    	public var name: String?
    	internal var isPet: Bool?
    	fileprivate var state: String?
    	private var age: Int? = 0
    
    	override init() {
        	self.name = "no name"
        	self.age = 0
        	self.isPet = true
        	self.state = "dead"
    	}
    
    	public func getName() -> String {
        	return self.name!
    	}
    
    	internal func setName(name: String?) {
        	self.name = name
    	}
	}
	
对应UML类图：  

![animal类图](AnimalUML类图.png)      

* 类名：粗体，如果是类是抽象类则类名显示为斜体！   
* 属性：   

> 可见性 名称：类型[=默认值]  

可见性一般为public、private和protected，在类图分别用+、-和#表示，在Swift中没有与protected完全对应的可见控制，因此选用的是internal对应为#；名称为属性的名称；类型为数据类型；默认值如变量 age默认值为0。    

* 方法：   

> 可见性 名称（参数列表 参数1，参数2） ：返回类型  

可见性如上名称表达式的介绍，名称就是方法名，参数列表是可选的项，多参数的话参数直接用英文逗号隔开；返回值也是个可选项，返回值类型可以说基本的数据类型、用户自定义类型和void。如果是构造方法，则无返回类型！

#### 4 类与类之间的关系表达   

类图中类与类之间的关系主要由：继承、实现、依赖、关联、聚合、组合这六大类型。表示方式如下图：   

![UML类与类之间关系](UML类与类之间关系.png)   


##### (1) 继承关系（Generalization/extends）

继承关系也叫泛化关系，指的是一个类（称为子类、子接口）继承另外的一个类（称为父类、父接口）的功能，并可以增加它自己的新功能的能力，继承是类与类或者接口与接口之间最常见的关系。   

继承用实线空心箭头表示，由子类指向父类。  

下面写两个子类，Fish和Cat分别继承自Animal。  

	class Fish: Animal {
	    
	    public var fishType: String?
	    func swim() {
	    }
	}
	
	class Cat: Animal {
	    public var hasFeet: Bool?
	    func playToy(doll:Doll) {
	        doll.toyMoved()
	    }
	}

![](Generalization.png)   

##### (2) 实现关系（implements）  

指的是一个class类实现interface接口（可以是多个）的功能；实现是类与接口之间最常见的关系；在Java中此类关系通过关键字implements明确标识，在iOS中我将其理解成代理的实现。   

写一个洋娃娃类Doll，该类遵循了ToyAction协议，实现了玩具移动的方法。   

	protocol ToyAction {
	    func toyMoved() -> Void
	}
	
	class Doll: NSObject,ToyAction {
	    
	    public var body: Body?
	    public var cloth: Cloth?
	    
	    func toyMoved() {
	        //洋娃娃玩具动作具体实现
	    }
	}

![](impletion.png)  

##### (3) 依赖关系（Dependency）

可以简单的理解，就是一个类A使用到了另一个类B，而这种使用关系是具有偶然性的、、临时性的、非常弱的，但是B类的变化会影响到A；比如某人要过河，需要借用一条船，此时人与船之间的关系就是依赖；表现在代码层面，为类B作为参数被类A在某个method方法中使用。   

在我们的上述代码中Cat的playToy方法中参数引用了Doll，因此他们是依赖关系。  
    
![](indepency.png)  


##### (4) 关联关系（Association）

他体现的是两个类、或者类与接口之间语义级别的一种强依赖关系，比如我和我的朋友；这种关系比依赖更强、不存在依赖关系的偶然性、关系也不是临时性的，一般是长期性的，而且双方的关系一般是平等的、关联可以是单向、双向的；表现在代码层面，为被关联类B以类属性的形式出现在关联类A中，也可能是关联类A引用了一个类型为被关联类B的全局变量；   

写一个Person类，他拥有一个宠物猫，他们之间的关系是关联。   

	class Head: NSObject {
	    
	}
	
	class Person: NSObject {
	    public var pet: Cat?
	    public var head: Head?
	}

![](association.png)    


##### (5) 聚合关系（Aggregation） 

聚合是关联关系的一种特例，他体现的是整体与部分、拥有的关系，即has-a的关系，此时整体与部分之间是可分离的，他们可以具有各自的生命周期，部分可以属于多个整体对象，也可以为多个整体对象共享；比如计算机与CPU、公司与员工的关系等；表现在代码层面，和关联关系是一致的，只能从语义级别来区分； 

	class Cloth: NSObject {
	    
	}
	
	class Body: NSObject {
	    
	}
	
在上述代码中Doll由Body和Cloth组成，且即使失去Cloth，Doll也可以正常存在。   

![](aggregation.png)    

##### (6) 组合关系（Composition）  

组合也是关联关系的一种特例，他体现的是一种contains-a的关系，这种关系比聚合更强，也称为强聚合；他同样体现整体与部分间的关系，但此时整体与部分是不可分的，整体的生命周期结束也就意味着部分的生命周期结束；比如你和你的大脑；表现在代码层面，和关联关系是一致的，只能从语义级别来区分； 

上述代码中的Person拥有Head，并且这个整体和部分是不可分割的。  

![](composition.png)


最后来看看这个例子中的整体关系：

![](UML类图示例.png)   


其实理解了之后我们发现还是很简单的，学会了还厚就可以投入实践中了，举一个简单第三方库的类图例子，下图是Masonry的类图整理，可以看到项目结构很清晰的展示了出来。   

![](Masonry类图.png)     








  
 





  

  




​
​  
​
​


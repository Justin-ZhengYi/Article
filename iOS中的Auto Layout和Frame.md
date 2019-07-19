### iOS中的Auto Layout和Frame
#### 1 iOS常用的布局方式 
iOS中界面有三种布局方式：Frame，Autoresizing Masks和Auto Layout。  

一般而言Frame是最随心所欲的，你可以做你任何想要的改变，但是同时也是最繁琐的，一旦布局发生改变你要更改所有的相关视图的Frame。Autoresizing Masks定义了父视图frame变化时子视图的frame如何变化，这简化了视图外部布局变化时内部视图变化的工作量。Auto Layout使用约束来表示视图间的相应关系，不论是外部变化还是内部变化，视图都可以动态相应，大大简化了我们的布局方式。    

以上是三种布局方式的主要特点，但也不绝对，比如使用Frame的时候我们也会根据其他视图的left，right去编写布局，这样也有了视图间的相互关系的意味。使用Auto Layout时的leading，trailing很多时候都是一个常量并不符合我们的要求，还要在视图加载或者变化过程中去修改值才能达到动态响应变化的效果。所以使用哪种布局方式好并没有一个定论，而且我也支持在特定的情况下使用有优势的布局方式。这样就造成了对于生命力较长的项目会出现多种方式并存的局面，本文章主要讲对于Frame和Auto Layout混用情况下的一些避坑策略。    

#### 2 视图加载过程   
要混用Frame和Auto Layout，必须要搞清楚他们的作用时间和有效范围，这和视图的加载过程有很大的关系。我们在controller视图上面用auto layout添加了一个subview，并在各个系统调用声明周期方法中做了打印处理，结果如下：  
 
	viewDidLoad   
	viewWillAppear 
	viewWillLayoutSubviews(1)  
	-updateConstraints   
	updateViewConstraints
	viewWillLayoutSubviews(2)
	viewDidLayoutSubviews   
	-layoutSubviews   
	-drawRect  
	viewDidAppear   

其中前面带有“-”符号的表示的是subview视图的打印。对于viewWillLayoutSubviews出现的顺序根据使用的布局方式会有所区别，使用Frame时出现为viewWillLayoutSubviews(1)位置，使用Auto Layout时出现在viewWillLayoutSubviews(2)位置。    

Auto Layout的作用范围是从第三个方法-updateConstraints开始直到viewDidLayoutSubviews完成，在这期间系统是通过view上的约束来计算view上的布局。一般我们都是在viewDidLoad里面编写页面布局代码，如果这时对一个视图同时使用了Auto Layout和Frame，我们会发现frame无效，就是因为这时布局是按照约束来计算的。如果视图的frame在布局过程中发生了改变（比如initWithFrame，使用xib都会给予初始值，但实际布局时我们可能想要更改），这时如果想获取视图的准确frame值，在viewWillAppear中是不行的，只能在自动布局生效后获取到，即我们可以再viewDidLayoutSubviews和viewDidAppear中获取frame的准确值。由上述结果也可以得出，系统的布局是优先使用Auto Layout的，但是布局的最终结果却是将约束转化成视图的frame，理解了这一点对于布局方式的选用也很重要。   

要想修改布局，必须要在Auto Layout结束之后才会起作用，否则会被系统将我们的布局按照Auto Layout重新刷新。从上面我们可以看到，我们可以在子视图的-layoutSubviews和-drawRect方法里面修改子视图的布局，但是一般视图为了重用不会在自己的类里面将frame写死，这样我们只有通过controller的viewDidLayoutSubviews和viewDidAppear方法修改视图的frame。但是这样也有问题，在viewDidAppear里面修改布局，我们可以看到一个明显的延迟，系统调用次序使然，我们无法去改变。

所以我的建议就是不要对同一个视图同时使用Frame和Auto Layout去控制同一个视图。如果一个视图使用了某种布局方式，那么尽量保持统一中方式改变布局。   

#### 3 不同布局方式混用  
不是说可以混用吗，怎么又突然说对同一个视图使用一种方式？那我们来定义一下混用方式： 父视图A包含子视图B、C，子视图B包含子视图B1、B2，子视图C包含子视图C1和C2。如图：  
![](iOS布局视图层级.png)

对于B和C我们可以使用Frame的方式布局其在父视图A中的位置和大小，但是对于B1和B2，我们可以使用Auto Layout的方式来布局其在父视图B中的尺寸和位置关系。**这就是我理解的混用，对于同一层级的视图使用相同的布局方式，对于不同层级的使用可以使用其他布局方式。**     

那么它的意义是什么呢？不同view的复杂度决定了我们采取哪种方式来对其布局，我们在开发中有可能会遇到某些流式的布局，我们用Auto Layout可以行云流水，但是对于某个视图当中极其复杂的小控件的布局，Auto Layout显然要写不少的一大段，当两者组合时也就是混用的意义所在。

#### 4 使用过程中的坑
   
（1） 约束变化后视图更新   

有的时候我们更改约束后发现视图并没有变化，这是因为约束改变后并没有触发视图重新布局。如果要重新布局，可以使用layoutIfNeeded()立即更新视图布局，使用setNeedsLayout()在下个绘图周期中触发布局更新。这两个方法都会触发layoutSubviews()方法。   

（2）不要启用NSAutoresizingMaskLayoutConstraint   
  
苹果iOS6之后推出了Auto Layout技术，但是还想兼容原来的autoresizing，所以它将autoresizing转化为了约束，转化后的约束类型就是NSAutoresizingMaskLayoutConstraint，这就有可能导致系统为我们添加了许多我们无法预料的约束。当我们添加的约束不，但是视图显示在界面上没有任何问题，这是NSAutoresizingMaskLayoutConstraint帮我们自动补齐了约束。在此情况下有可能出现frame不正确。所以我们在混用时不要开启此属性，即设置其为NO。在storyboard上NSAutoresizingMaskLayoutConstraint默认为开启，纯代码默认为关闭。  

**总结：**   
（1）使用Auto Layout时NSAutoresizingMaskLayoutConstraint设置为NO，设置完整清晰的约束。   
（2）对同一层级视图使用相同布局方式，对非同一层级视图可以使用不同布局方式。对同一视图在同一布局时间段内不要同时使用不同的布局方式。  
（3）搞清视图布局、绘制步骤，区分清楚Frame和Auto Layout的有效范围，在合适的时机进行视图布局变化、更新和frame获取。   




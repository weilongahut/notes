
###Effective Java中文版 第二版###

####第8条：覆盖equals时请遵守通用约定####
Object提供的equals方法的判断逻辑是每个实例只和自身相等，在Object的子类中，满足一下条件之一，Object.euqals（）方法的逻辑也是期望的结果：

- 类的每个实例本质上都是唯一的，代表活动实体而不是值（value）的类是如此，如Thread
- 不关心类是否提供了“逻辑相等（logical equality）”的测试功能，例如，`java.util.Random`覆盖了equals，以检查两个Random实例是否产生相同的随机数序列，但是设计者并不认为客户需要或者期望这样的功能，事实上不会有这种需求，这种情况下，Object继承得到的equals实现已经足够
- 超类已经覆盖了equals，从超类继承过来的行为对于子类也是合适的。大多数的Set实现都从AbstractSet继承equals实现，List实现从AbstractList继承equals实现，Map实现从AbstractMap继承equals实现
- 类时私有的，或者是包级私有的，可以确定它的equals方法永远不会被调用。这种情况下，也应该覆盖equals方法，防止被意外调用
	
		@Override public boolean equals(Object o){
			throw new AsssertionError(); //method is never called
		} 

<Strong>如果一个类有自己特有的逻辑相等的概念，不同于对象等同的概念，而且超类还没有覆盖equals以实现期望的行为，这时我们就需要覆盖equals方法</Strong>

根据Java SE6的规范，equals方法实现了等价关系（equivalent relation）：

- 自反性（reflexive）
- 对称性（symmetic）
- 传递性（transitive）
- 一致性（consistent）

面向对象语言中关于等价关系的一个基本问题：<Strong> 无法在扩展可实例化的类的同是，既增加新的值组件，同时又保留equals约定</Strong>
有一种方法，在equa方法中用getClass测试替代instanceOf测试，可以扩展可实例化的类和增加新的值组件，但是要注意这种方法只有当对象具有相同的实现时，才能使对象等同。

	//Broken -violates Liskov substitution principle
	@Override
	public boolean equals(Object o){
		if(o == null || o.getClass() != getClass()){
			return false;
		}
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
**里氏替换原则（Liskov substitution principle）**认为，一个类型的任何重要属性也将适用于它的子类型，因此为该类型编写的任何方法，在它的子类型上也应该同样运行得很好。

实现高质量equals方法的诀窍:

- 1 使用==操作符检查“参数是否为这个对象的引用”，
- 2 使用instanceof操作符检查“参数是否为正确的类型”
- 3 把参数转换成正确的类型
- 4 对于该类中的每个关键（significant）域，检查参数中的域是否与该对象中对应的域相匹配

注意：对于float和double类型的域，需要使用特殊的方法进行比较，Float.compare和Double.compare，因为会有Float.Nan，-0.0f等类似的值

- 5 当你编写完成了equals方法之后，应该问自己三个问题：它是否是对称的、传递的、一致的？

其他需要注意的：

- 覆盖equals时总要覆盖hashCode
- 不要企图让equals方法过于智能
- 不要将equals声明中的Object对象替换为其他的类型

#### 第9条 覆盖equals时总要覆盖hashCode ####
计算hashCode的方法：

- 1 把某个非零的常数值，比如17，保存在一个名为result的int类型变量中
- 2 对于对象中每个关键域f（指equals方法中涉及的每个域），完成以下步骤：
	- a. 为该域计算int类型的hashCode：
		- 如果该域是boolean类型，则计算（f?1:0）
		- 如果该域是byte、char、short或者int类型，则计算(int)f
		- 如果该域是long类型，则计算（int）(f ^ (f >>>32))
		- 如果该域是float类型，则计算机Float.floatToIntBits(f)
		- 如果该域是double类型，则计算Double.doubleToLongBits（f),然后再以此long值计算（int）(f ^ (f >>> 32))
		- 如果该域是一个对象引用，并且该类的equals方法通过递归地调用equals方式来比较这个域，则同样为这个域递归地调用hashCode方法，如果需要更复杂的比较，则为这个域计算一个“范式（canonical representation）”，然后针对这个范式调用hashCode。如果这个域为null，则返回0，也可以返回某个常数
		- 如果该域是一个数组，则要把每一个元素当做单独的域来处理，也就是说递归地应用上述规则，对每个重要的元素计算一个散列码，然后加在一起，如果数组域中的每个元素都很重要，可以利用Arrays.hashCode方法
		
	- b. 按照下面的公式，把步骤2.a中计算的散列值c合并到result中：
	
		result = 31 * result + c
- 3 返回result
- 4 写完了hashCode方法之后，问问自己“相等的实例是否都具有相等的散列码”，**要编写单元测试来验证你的推断**！！！

**Always remember：不要试图从散列码计算中排除掉一个对象的关键部分来提高性能**


#### 第10条： 始终要覆盖toString ####
JavaSE6约定toString方法应该返回一个**简洁的，但信息丰富，并且易于阅读的表达形式**的字符串，建议所有的子类都覆盖这一方法。

toString方法应该返回对象中包含的所有值得关注的信息

**无论是否指定格式，都应该在文档中明确地表明你的意图，并且要为toString返回值中包含的所有信息提供一种编程式的访问途径**

#### 第11条：谨慎地覆盖clone ####

Cloneable接口不包含任何方法，它的作用是决定Object中受保护的clone方法的实现行为：如果一个类实现了Cloneable接口，Object的clone方法就返回该对象的逐域拷贝，否则就抛出CloneNotSupportedException异常。这是接口的一种极端非典型的用法，也不值得仿效。

实际上，**clone方法就是另外一个构造器，你必须确保它不会伤害到原始的对象，并确保正确地创建被克隆对象中的约束条件（invariant）**


如果扩展一个实现了Cloneable接口的类，那么除了实现一个行为良好的clone方法外，没有别的选择。否则，**最好提供某些其他的途径来代替对象拷贝，或者干脆不提供这样的功能**，另一个实现对象拷贝的好办法是提供一个**拷贝构造器（copy constructor）或拷贝工厂（copy factory）**


#### 第12条：考虑实现Comparable接口 ####

对于一个存在非常明显的内在排序关系的值类，应该实现Comparable接口:
	
	public interface Comparable<T> {
		int compareTo(T t);
	}

compareTo方法的通用约定：

- 实现者必需确保所有的x和y都满足sgn(x.compareTo(y)) == -sgn(y.compareTo(x))。这也暗示着，当且仅当y.compareTo(x)抛出异常时，x.compareTo（y）才必须抛出异常
- 实现者还必须确保这个比较关系是可传递的：（x.compareTo(y) > 0 && y.compareTo(z) > 0）暗示着x.compareTo(z) > 0
- 最后，实现者必需确保x.compareTo（y） == 0 暗示着所有的z都满足sgn（x.compareTo(z)）== sgn(y.compareTo(z))
- 强烈建议（x.compareTo(y) == 0）== (x.equals(y)), 但是这并非必要。一般来说，任何实现了Comparable接口的类，如果违反了这个条件，都应该明确予以说明。推荐使用这样的说法：“注意：该类具有内在排序功能，但是和equals不一致”

#### 第13条 使类和成员的可访问性最小化
**要区别设计良好的模块与设计不好的模块，最重要的因素在于，这个模块对于外部模块的其他模块而言，是否隐藏了其内部数据和其他实现细节**
**一个实现了Serializable接口的类，可能会导致私有域被泄露（leak）， 因为在反序列化时可以解析到这些私有域**

**实例域绝不可以是共有的，否则，即使这是一个final的域，也会导致丧失切换到一种新的内部数据表示法的灵活性**

对于静态域，除了基本类型和指向不可变对象的引用，尽量也不要公有化。

长度非零的数组总是可变的，所以不应该返回该数组的引用，可以在返回它的备份。

#### 第14条 在公有类中使用访问访问法而非公有域

公有类不应该暴露可变的域，即便是不可变域也只是危害性相对小一点而已。**但是，有时候会需要用包级私有的或者私有的嵌套类来暴露域，无论这个类是可变的还是不可变的**


#### 第15条 使可变性最小化
为了使类成为不可变，要遵循5条规则：

- 1 不要提供任何会修改对象状态的方法（mutator）
- 2 保证类不会被扩展，其中一个做法是声明为final
- 3 使所有的域都成为final的
	- 不可变对像更为简单
	- 不可变对象本质上线程安全的，他们不要求同步
	- 不可变对象可以自由地被分享
	- 不仅可以共享不可变对象，甚至也可以共享他们的内部信息
- 4 使所有的域都成为私有的
- 5 确保对任何可变组件的互斥访问

除非有很好的理由要让类成为可变的类，否则就应该是不可变的，不可变的类有很多优点，但是在特定的情况下存在潜在的性能问题。大多数情况下，类无法做成不可变的，这时候应该尽量限制它的可变性。**除非有必要让域变成是非final的，否则要使每个域都是final的**

#### 第16条 复合优先于继承

继承（inheritance）是实现代码重用的有力手段，然而它并不是完成这项 工作的最佳工具。使用不当会导致软件变得很脆弱。**普通的具体类（concrete）跨越包边界的继承，是非常危险的**

与方法调用不同，**继承打破了封装性(encapsulation)**

复合（composition）是在新的类中增加一个私有域，不扩展现有类，它引用现有类的一个实例，新类中的每个实例方法都可以调用被包含的现有类实例中对应的方法，并返回它的结果，这被成为转发（forwarding）

只有当子类真正是超类的子类型（subtype）时，才适合用继承。在Java平台类中，有一些明显违反这条原则的地方，比如Stack并不是Vector，所有Stack不应该继承Vector，同样的属性列表也不是散列表，所以Properties不应该扩展Hashtable。


#### 第17条 要么为继承而设计，并提供文档说明，要么就禁止继承

该类的文档必须精确地描述覆盖每个方法所带来的影响，换句话说，该类必须有文档说明它可覆盖（overridable）的方法的自用性（self-use）。
	
关于程序文档有句格言：**好的API文档应该描述一个给定的方法做了什么工作而不是描述它是如何做到的。**

为了继承而进行的设计不仅仅涉及自用模式的文档涉及，为了使程序员能够写出更加有效的子类，而不需承受不必要的痛苦，类必须通过某种形式提供适当的钩子（hook），以便能够进入到它的内部工作流程中，这种形式可以是精心选择受保护的（protected）方法，也可以是受保护的域。

对于为继承而设计的类，唯一的测试方法就是编写子类。对于文档中所说明的自用模式（self-use pattern）,以及对于其受保护方法和域中所隐含的实现策略，你实际上已经做出了永久的承诺。这些承诺使得你在后续的版本中提高这个类的性能或者增加新功能都变得非常困难，甚至不可能。因此，**必须在发布之前编写子类对类进行测试**

为了允许继承，类还必须遵守其他一些约束。**构造器绝对不能调用可被覆盖的方法，无论是直接调用还是间接调用**

如果决定在一个为继承而设计的类中实现Cloneable或者Serializable接口，就应该意识到，因为clone和readObject方法在行为上非常类似于构造器，所以类似的限制规则也是使用的：无论是clone还是readObject，都不可以调用可覆盖的方法，不管是以直接还是间接的方式。对于readObject方法，覆盖版本的方法将在子类的状态被反序列化（deserialized）之前裕兴，而对于clone方法，覆盖版本的方法则是在子类的clone方法有机会修正被克隆对象的状态之前先被运行。

对于那些并非为了安全地进行子类化而设计和编写文档的类，要禁止子类化。有两种方法可以禁止子类化：

- 一种是把这个类声明为final的
- 另外一种是吧所有的构造器都变成私有的，或者包级私有的，并增加一些共有的静态工厂来代替构造器。

#### 第18条 接口优于抽象类

Java程序设计语言提供了两种机制，可以用来定义云溪多个实现的类型：**接口和抽象类**。二者的主要去呗在于，抽象类允许包含某些方法的实现，但是接口则不允许。一个更为重要的区别在于，为了实现由抽象定义的类型，类必须成为抽象类的一个子类。任何一个类，只要它定义了所有必要的方法，并且遵守通用的约定，他就被允许实现一个接口，而不管这个接类是处于类层次（class hierarchy）的哪个位置。

接口的优点：

- 现有的类可以很容易被更新，以实现新的接口
- 接口是定义mixin(混合类型)的理想选择
- 接口允许我们构造非层次接口的类型框架


虽然接口不允许包含方法的实现，但是使用接口来定义类型并不妨碍你为程序员提供实现上的帮助。**通过对你导出的每个重要接口都提供一个抽象的股价实现（skeletal implementation）类，把接口和抽象类的优点结合起来**。接口的作用仍然是定义类型，但是骨架实现类接管了所有与接口实现相关的工作。

此外，骨架实现类仍然能够有助于接口的实现，实现了这个接口的类可以把对于接口方法的调用，转发到一个内部私有类的实例上，这个内部私有类扩展了骨架实现类。这种方法被称作模拟多重继承（simulated multiple inheritance）。

#### 第19条 接口只用于定义类型
当类实现接口时，接口就充当了可以引用这类的实例的类型（type）。因此，类实现了接口，就表明客户端可以对这类的实例实时某些动作。为了任何其他目的而定义接口是不恰当的。

有一种接口被称作常量接口（constant interface），他是接口的一种特殊情况，这种接口没有包含任何方法，它只包含静态的final域，每个域都导出一个常量。使用这些常量的类实现这个接口，以避免用类名来修饰常量名。

	//Constant interface antipattern -do not use
	public interface PhysicalConstants{
		//Avogadro's member
		static final double AVOGADROS_NUMBER = 0232323;
		
		//Boltzmann constant
		statit final double BOLTZMANN_CONSTANT = 1.2323e1;
	}
	
常量接口模式对接口的不良使用！ 

导出常量的几种方法：
- 1 如果这些常量与某个现有的类或者接口紧密相关，就应该把这些常量添加到这个类或者接口中
- 2 如果这些常量最好被看做枚举类型的成员，就应该用枚举类型（enum type）来导出这些常量
- 3 使用不可实例化的工具类（utility class）来导出这些常量。

#### 第20条 类层次优于标签类

标签类tag：

	//Tagged class - vastly inferior to a class hierarchy!
	class Figure{
		enum Shape { RECTANGLE, CIRCLE };
		
		final Shape shape;
		
		doubel length;
		double width;
		
		double radius;
		
		Figure(double radius){
			shape = Shape.CIRCLE;
			this.radius = radius;
		}
		
		Figure(double length, double width){
			shape = Shape.RECTANGLE;
			this.length = lenght;
			this.width = width;
		}
		
		double area(){
			switch(shape){
				case RECTANGLE:
					return length * width;
				case CIRCLE:
					return Math.PI * (radius * radius); 
				default :
					throw new AssertionError();
			}
			
		}
	
	}
	
	
这种标签类（tagged class）有着许多缺点，他们充斥着样板代码，包括枚举声明、标签域以及条件语句。可读性差。

通过子类型化（subtyping）将标签类转化为类层次，首先要为标签类中的每个方法都定义一个包含抽象方法的抽象类，这每个方法的行为都依赖于标签值。

#### 第21条 用函数对象表示策略

有些语言支持函数指针（function pointer）、代理（delegate）、lambda表达式（lambda expression），或者支持类似的机制，允许程序把“调用特殊函数的能力”存储起来并传递这种能力。

Java没有提供这种函数指针，但是可以通过对象引用来实现同样的功能，调用对象上的方法通常是执行该对象的（that object）某项操作。然而我们也可以定义这样的一种对象，它的方法执行其他对象（other objects）上的操作。这样的实例被称为函数对象（function object），如：
	
	class StringLengthComparator{
		public int comparable(String s1, String s2){
			return s1.length - s2.lenght;
		}
	}
	

作为典型的具体策略类，StringLengthComparator是无状态（stateless）的，因为它没有域。所以这个类的所有实例在功能上都是相互等价的。

**再设计具体策略类时，往往要先定义一个策略接口（strategy interface）**

#### 第22条 优先考虑静态成员类

嵌套类（nested class）是被定义在另一个类的内部的类。嵌套类存在的目的应该只是为它的外围类（eclosing class）提供服务。如果嵌套类将来可能会用于其他环境中，它就应该是顶层类（top-level class）。嵌套类有四种：

- 静态成员类，static member class
- 非静态成员类 nonstatic member class
- 匿名类, anonymous class
- 局部类， local class

当非静态成员类的实例被创建的时候，它和外围实例之间的关联关系也随之被建立起来，而且，这种关联关系以后不能被修改。通常，当在外围类的某个实例方法的内部调用非静态成员类的构造器时，这种关联关系被自动建立起来。使用表达式enclosingInstance.new MemberClass(args)来手工建立这种关联关系也是有可能的，但是很少使用。这种关联关系需要消耗费静态成员类实例的空间，并且增加了构造的时间开销。

四种不同的嵌套类，各有用途：如果一个嵌套类需要在单个方法之外仍然是可见的，或者它太长了不适合放在方法内部，就应该使用成员类。如果成员类的每个实例都需要一个指向其外围类实例的引用，就要把成员类做成非静态的，否咋就做成静态的。假设这个嵌套类属于一个方法的内部，如果你只需要在一个地方创建实例，并且已经有了一个预置的类型可以说明这个类的特征，就要把它做成匿名类，否则就做成局部类。


### 第5章 泛型
#### 第23条 请不要在新代码中使用原生类型

如果使用原生类型，就失掉了泛型（generic）在安全性和表述性方面的所有优势。

#### 第24条 消除unchecked警告

如果无法消除警告，同时可以证明引起警告的代码是类型安全的，（只有在这种情况下才）可以用一个@SuppressWarning（“unchecked”）注解来禁止这条警告。

每当使用SuppressWarning的时候都要添加一条注释，说明为什么这样做是安全的

#### 第25条 列表优先于数组

数组和泛型的区别在于：

- 数组是协变的（covariant），泛型是不可变的（invariant）
- 数组是具体化的，数组会在运行时才知道并检查他们的元素类型约束，泛型则是通过擦除（erasure）来实现类型的约束

#### 第26条 优先考虑泛型

使用泛型比使用需要在客户端代码中进行转换的类型更加安全，也更加容易。在设计新类型的时候，要确保他们不需要这种转换就可以使用。这通常以为这要把类做成是泛型的。只要时间允许，就把现有的类型都泛型化。

#### 第27条 优先考虑泛型方法

泛型方法的一个显著特征是，无需明确指定类型参数的值，不像调用泛型构造器的时候是必须指定的。编译器通过检查方法参数的类型来设计计算类型参数的值。

泛型方法就像泛型一样，使用起来比要求客户端转换输入参数并返回值的方法来的更加安全，也更容易。就像类型一样，你应该确保新方法可以不用转换就能使用，这通常意味着要将它们泛型化。

#### 第28条 利用有限制通配符来提升API的灵活性

在API中使用通配符类型虽然比较需要技巧，但是使API变得灵活得多。如果编写的是将广泛使用的类库，则一定要适当地利用通配符类型。

#### 第29条 优先考虑类型安全的异构容器

集合API说明了泛型的一般用法，限制你每个容器只能有固定数目的类型参数。可以通过将类型参数放在键上而不是容器上来避开这一限制。对于这种类型安全的异构容器，可以用class对象作为键。以这种方式使用的Class对象称作类型令牌。你也可以使用定制的键类型。

### 第6章 枚举和注解

#### 第30条 用enum代替int常量









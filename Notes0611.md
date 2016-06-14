
##Spring in Action##

###Spring AOP###

Spring‘s AOP configuration elements enable non-invasive declaration of aspects

- <aop:advisor>
- <aop:after>
- <aop:after-returning>
- <aop:after-throwing>
- <aop:around>
- <aop:aspect>
- <aop:aspectj-autoproxy>
- <aop:before>
- <aop:config>
- <aop:declare-parents>
- <aop:pointcut>

----
Declaaring before and after adivce
	
	<aop:config>
		<aop:aspect ref="audience">
			<aop:before
				pointcut="execution(** concert.Performance.perform(..))"
				method="silenceCellPhones" />
			<aop:before
				pointcut="execution(** concert.Performance.perform(..))"
				method="takeSeats" />
			<aop:after-returning
				pointcut="execution(** concert.Performance.perform(..))
				method="applause" />
			<aop:after-throwing
				pointcut="execution(** concert.Performance.perform(..))"
				method="demandRefund" />
		</aop:aspect>
	</aop:config>



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


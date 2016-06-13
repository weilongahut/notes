
# Spring in Action

##Spring AOP

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


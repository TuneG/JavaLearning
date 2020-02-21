#### 抽象类

建立了一种基本形式，使我们能定义在所有衍生类里“通用”的一些东西。为阐述这个观念，另一个方法是把Instrument称为“抽象基础类”（简称“抽象类”）。若想通过该通用接口处理一系列类，就需要创建一个抽象类。对
所有与基础类声明的签名相符的衍生类方法，都可以通过动态绑定机制进行调用（然而，如果方法名与基础类相同，但自变量或参数不同，就会出现过载现象（重写），那或许并非我们所愿意的）。
如果有一个象Instrument 那样的抽象类，那个类的对象几乎肯定没有什么意义。换言之，Instrument 的作
用仅仅是表达接口，而不是表达一些具体的实施细节。所以创建一个Instrument 对象是没有意义的，而且我
们通常都应禁止用户那样做。为达到这个目的，可令Instrument 内的所有方法都显示出错消息。但这样做会
延迟信息到运行期，并要求在用户那一面进行彻底、可靠的测试。无论如何，最好的方法都是在编译期间捕
捉到问题。
针对这个问题，Java 专门提供了一种机制，名为“抽象方法”。它属于一种不完整的方法，只含有一个声
明，没有方法主体。下面是抽象方法声明时采用的语法：
`abstract void X();`
***包含了抽象方法的一个类叫作“抽象类”。***

注意：

1. 如果一个类里包含了一个或多个抽象方法，类就必须指定成abstract（抽象）。否则，编译器会向我们报告一条出错消息。
2. 若一个抽象类是不完整的，那么一旦有人试图生成那个类的一个对象，编译器又会采取什么行动呢？由于不能安全地为一个抽象类创建属于它的对象，所以会从编译器那里获得一条出错提示。通过这种方法，编译器可保证抽象类的“纯洁性”，我们不必担心会误用它。
3. 如果从一个抽象类继承，而且想生成新类型的一个对象，就必须为基础类中的所有抽象方法提供方法定义。
   如果不这样做（完全可以选择不做），则衍生类也会是抽象的，而且编译器会强迫我们用abstract 关键字标
   志那个类的“抽象”本质。
4. 即使不包括任何abstract 方法，亦可将一个类声明成“抽象类”。如果一个类没必要拥有任何抽象方法，而
   且我们想禁止那个类的所有实例，这种能力就会显得非常有用。
   Instrument 类可很轻松地转换成一个抽象类。只有其中一部分方法会变成抽象方法，因为使一个类抽象以
   后，并不会强迫我们将它的所有方法都同时变成抽象。下面是它看起来的样子：

![image-20191204092117630](D:\sgmuserprofile\sqgdua\Desktop\image-20191204092117630.png)



下面是我们修改过的“管弦”乐器例子，其中采用了抽象类以及方法：

~~~java
//: Music4.java
// Abstract classes and methods
import java.util.*;

abstract class Instrument4 {
	int i; // storage allocated for each
	public abstract void play();
	public String what() {
		return "Instrument4";
	}
	public abstract void adjust();
}

class Wind4 extends Instrument4 {
	public void play() {
		System.out.println("Wind4.play()");
	}
	public String what() { return "Wind4"; }
	public void adjust() {
        
    }
}
class Percussion4 extends Instrument4 {
	public void play() {
		System.out.println("Percussion4.play()");
	}
	public String what() { 
        return "Percussion4"; 
    }
	public void adjust() {}
}
class Stringed4 extends Instrument4 {
	public void play() {
		System.out.println("Stringed4.play()");
	}
	public String what() { 
        return "Stringed4"; 
    }
	public void adjust() {}
}
class Brass4 extends Wind4 {
	public void play() {
		System.out.println("Brass4.play()");
	}
	public void adjust() {
		System.out.println("Brass4.adjust()");
	}
}
class Woodwind4 extends Wind4 {
	public void play() {
		System.out.println("Woodwind4.play()");
	}
	public String what() { 
        return "Woodwind4"; 
    }
}
public class Music4 {
// Doesn't care about type, so new types
// added to the system still work right:
	static void tune(Instrument4 i) {
		// ...
		i.play();
	}
	static void tuneAll(Instrument4[] e) {
		for(int i = 0; i < e.length; i++)
		tune(e[i]);
	}
	public static void main(String[] args) {
		Instrument4[] orchestra = new Instrument4[5];
		int i = 0;
		// Upcasting during addition to the array:
		orchestra[i++] = new Wind4();
		orchestra[i++] = new Percussion4();
		orchestra[i++] = new Stringed4();
		orchestra[i++] = new Brass4();
		orchestra[i++] = new Woodwind4();
		tuneAll(orchestra);
	}
} ///:~
~~~

可以看出，除基础类以外，实际并没有进行什么改变。
创建抽象类和方法有时对我们非常有用，因为它们使一个类的抽象变成明显的事实，可明确告诉用户和编译
器自己打算如何用它。



#### 接口

“interface”（接口）关键字使抽象的概念更深入了一层。我们可将其想象为一个“纯”抽象类。它允许创
建者规定一个类的基本形式：方法名、自变量列表以及返回类型，但不规定方法主体。接口也包含了基本数
据类型的数据成员，但它们都默认为static 和final。接口只提供一种形式，并不提供实施的细节。
接口这样描述自己：“对于实现我的所有类，看起来都应该象我现在这个样子”。因此，采用了一个特定接
口的所有代码都知道对于那个接口可能会调用什么方法。这便是接口的全部含义。所以我们常把接口用于建
立类和类之间的一个“协议”。有些面向对象的程序设计语言采用了一个名为“protocol”（协议）的关键
字，它做的便是与接口相同的事情。
为创建一个接口，请使用interface 关键字，而不要用class。与类相似，我们可在interface 关键字的前
面增加一个public 关键字（但只有接口定义于同名的一个文件内）；或者将其省略，营造一种“友好的”状
态。
为了生成与一个特定的接口（或一组接口）相符的类，要使用implements（实现）关键字。我们要表达的意
思是“接口看起来就象那个样子，这儿是它具体的工作细节”。除这些之外，我们其他的工作都与继承极为
相似。下面是乐器例子的示意图：

![image-20191204092810171](D:\sgmuserprofile\sqgdua\Desktop\image-20191204092810171.png)



具体实现了一个接口以后，就获得了一个普通的类，可用标准方式对其进行扩展。
可决定将一个接口中的方法声明明确定义为“public”。但即便不明确定义，它们也会默认为public。所以
在实现一个接口的时候，来自接口的方法必须定义成public。否则的话，它们会默认为“友好的”，而且会
限制我们在继承过程中对一个方法的访问——Java 编译器不允许我们那样做。
在Instrument 例子的修改版本中，大家可明确地看出这一点。注意接口中的每个方法都严格地是一个声明，
它是编译器唯一允许的。除此以外，Instrument5 中没有一个方法被声明为public，但它们都会自动获得
public 属性。如下所示：

~~~java
//: Music5.java
// Interfaces
import java.util.*;

interface Instrument5 {
// Compile-time constant:
	int i = 5; // static & final
// Cannot have method definitions:
	void play(); // Automatically public
	String what();
	void adjust();
}

class Wind5 implements Instrument5 {
	public void play() {
	System.out.println("Wind5.play()");
	}

	public String what() {
    	return "Wind5";
	}
    
	public void adjust() {
        
    }
}

class Percussion5 implements Instrument5 {
	public void play() {
		System.out.println("Percussion5.play()");
	}
	public String what() { 
        return "Percussion5"; 
    }
	public void adjust() {
        
    }
}

class Stringed5 implements Instrument5 {
	public void play() {
		System.out.println("Stringed5.play()");
	}
	public String what() {
        return "Stringed5"; 
    }
	public void adjust() {
        
    }
}

class Brass5 extends Wind5 {
	public void play() {
		System.out.println("Brass5.play()");
	}
	public void adjust() {
		System.out.println("Brass5.adjust()");
	}
}

class Woodwind5 extends Wind5 {
	public void play() {
		System.out.println("Woodwind5.play()");
	}
	public String what() { 
        return "Woodwind5"; 
    }
}

public class Music5 {
// Doesn't care about type, so new types
// added to the system still work right:
	static void tune(Instrument5 i) {
// ...
		i.play();
	}
	static void tuneAll(Instrument5[] e) {
		for(int i = 0; i < e.length; i++)
		tune(e[i]);
	}
	public static void main(String[] args) {
		Instrument5[] orchestra = new Instrument5[5];
		int i = 0;
		// Upcasting during addition to the array:
		orchestra[i++] = new Wind5();
		orchestra[i++] = new Percussion5();
		orchestra[i++] = new Stringed5();
		orchestra[i++] = new Brass5();
		orchestra[i++] = new Woodwind5();
		tuneAll(orchestra);
	}
} ///:~
~~~



#### Java的“多重继承”

接口只是比抽象类“更纯”的一种形式。它的用途并不止那些。由于接口根本没有具体的实施细节——也就
是说，没有与存储空间与“接口”关联在一起——所以没有任何办法可以防止多个接口合并到一起。这一点
是至关重要的，因为我们经常都需要表达这样一个意思：“x 从属于a，也从属于b，也从属于c”。在C++
中，将多个类合并到一起的行动称作“多重继承”，而且操作较为不便，因为每个类都可能有一套自己的实
施细节。在Java 中，我们可采取同样的行动，但只有其中一个类拥有具体的实施细节。所以在合并多个接口
的时候，C++的问题不会在Java 中重演。如下所示：

![image-20191204093133179](D:\sgmuserprofile\sqgdua\Desktop\image-20191204093133179.png)





代码剩余的部分按相同的方式工作。我们可以自由决定上溯造型到一个名为Instrument5 的“普通”类，一
个名为Instrument5 的“抽象”类，或者一个名为Instrument5 的“接口”。所有行为都是相同的。事实
上，我们在tune()方法中可以发现没有任何证据显示Instrument5 到底是个“普通”类、“抽象”类还是一
个“接口”。这是做是故意的：每种方法都使程序员能对对象的创建与使用进行不同的控制。

在一个衍生类中，我们并不一定要拥有一个抽象或具体（没有抽象方法）的基础类。如果确实想从一个非接
口继承，那么只能从一个继承。剩余的所有基本元素都必须是“接口”。我们将所有接口名置于implements
关键字的后面，并用逗号分隔它们。可根据需要使用多个接口，而且每个接口都会成为一个独立的类型，可
对其进行上溯造型。下面这个例子展示了一个“具体”类同几个接口合并的情况，它最终生成了一个新类：

~~~java
//: Adventure.java
// Multiple interfaces
import java.util.*;

interface CanFight {
	void fight();
}

interface CanSwim {
	void swim();
}

interface CanFly {
	void fly();
}

class ActionCharacter {
	public void fight() {}
}

class Hero extends ActionCharacter implements CanFight, CanSwim, CanFly {
	public void swim() {}
	public void fly() {}
}

public class Adventure {
	static void t(CanFight x){ 
        x.fight(); 
    }
	static void u(CanSwim x) { 
        x.swim(); 
    }
	static void v(CanFly x) { 
        x.fly(); 
    }
	static void w(ActionCharacter x) { 
        x.fight(); 
    }
	public static void main(String[] args) {
		Hero i = new Hero();
		t(i); // Treat it as a CanFight
		u(i); // Treat it as a CanSwim
		v(i); // Treat it as a CanFly
		w(i); // Treat it as an ActionCharacter
	}
} ///:~
~~~

从中可以看到，Hero 将具体类ActionCharacter 同接口CanFight，CanSwim 以及CanFly 合并起来。按这种
形式合并一个具体类与接口的时候，具体类必须首先出现，然后才是接口（否则编译器会报错）。
请注意fight()的签名在CanFight 接口与ActionCharacter 类中是相同的，而且没有在Hero 中为fight()提
供一个具体的定义。接口的规则是：我们可以从它继承（稍后就会看到），但这样得到的将是另一个接口。
如果想创建新类型的一个对象，它就必须是已提供所有定义的一个类。尽管Hero 没有为fight()明确地提供
一个定义，但定义是随同ActionCharacter 来的，所以这个定义会自动提供，我们可以创建Hero 的对象。
在类Adventure 中，我们可看到共有四个方法，它们将不同的接口和具体类作为自己的自变量使用。创建一
个Hero 对象后，它可以传递给这些方法中的任何一个。这意味着它们会依次上溯造型到每一个接口。由于接
口是用Java 设计的，所以这样做不会有任何问题，而且程序员不必对此加以任何特别的关注。
注意上述例子已向我们揭示了接口最关键的作用，也是使用接口最重要的一个原因：能上溯造型至多个基础
类。使用接口的第二个原因与使用抽象基础类的原因是一样的：防止客户程序员制作这个类的一个对象，以
及规定它仅仅是一个接口。这样便带来了一个问题：到底应该使用一个接口还是一个抽象类呢？若使用接
口，我们可以同时获得抽象类以及接口的好处。所以假如想创建的基础类没有任何方法定义或者成员变量，
那么无论如何都愿意使用接口，而不要选择抽象类。事实上，如果事先知道某种东西会成为基础类，那么第
一个选择就是把它变成一个接口。只有在必须使用方法定义或者成员变量的时候，才应考虑采用抽象类。

#### Extends&&implements的区别

Extends可以理解为全盘继承了父类的功能。implements可以理解为为这个类附加一些额外的功能；interface定义一些方法,并没有实现,需要implements来实现才可用。extends可以继承一个接口,但仍是一个接口,也需要implements之后才可用。对于class而言，Extends用于(单)继承一个类（class），而implements用于实现一个接口(interface)。

> interface的引入是为了部分地提供多继承的功能。
> 在interface中只需声明方法头，而将方法体留给实现的class来做。
> 这些实现的class的实例完全可以当作interface的实例来对待。
> 在interface之间也可以声明为extends（多继承）的关系。
> 注意一个interface可以extends多个其他interface。

在类的声明中，通过关键字extends来创建一个类的子类。一个类通过关键字implements声明自己使用一个或者多个接口。extends 是继承某个类, 继承之后可以使用父类的方法, 也可以重写父类的方法;implements 是实现多个接口, 接口的方法一般为空的, 必须重写才能使用.

extends是继承父类，只要那个类不是声明为final或者那个类定义为abstract的就能继承，JAVA中不支持多重继承，但是可以用接口 来实现，这样就要用到implements，继承只能继承一个类，但implements可以实现多个接口，用逗号分开就行了;比如 `class A extends B implements C,D,E`;

学了好久,今天终于明白了implements（实现接口就是在接口中定义了方法，这个方法要你自己去实现，接口可以看作一个标准，比如定义了一个动物的接口，它里面有吃（eat()）这个方法，你就可以实现这个方法implements，这个方法是自己写，可以是吃苹果，吃梨子，香蕉，或者其他的。IMPLEMENTS就是具体实现这个接口。）,其实很简单,看下面几个例子就ok啦：

```java
public inerface Runner
{
  int ID = 1;
  void run ();
}
interface Animal extends Runner
{
  void breathe ();
}
class Fish implements Animal
{
  public void run ()
{
   System.out.println("fish is swimming");
}
public void breather()
{
   System.out.println("fish is bubbing");   
}
}
abstract LandAnimal implements Animal
{
  public void breather ()
{
   System.out.println("LandAnimal is breathing");
}
}
class Student extends Person implements Runner
{
   ......
   public void run ()
    {
         System.out.println("the student is running");
    }
   ......
}
interface Flyer
{
  void fly ();
}
class Bird implements Runner , Flyer
{
  public void run ()
   {
       System.out.println("the bird is running");
   }
  public void fly ()
   {
       System.out.println("the bird is flying");
   }
}
class TestFish
{
  public static void main (String args[])
   {
      Fish f = new Fish();
      int j = 0;
      j = Runner.ID;
      j = f.ID;
   }
}
```

接口实现的注意点：
a.实现一个接口就是要实现该接口的所有的方法(抽象类除外)。
b.接口中的方法都是抽象的。
c.多个无关的类可以实现同一个接口，一个类可以实现多个无关的接口。

------

**extends与implements的不同**: 一个类通过关键字implements声明自己使用一个或者多个接口。在类的声明中，通过关键字extends来创建一个类的子类。

```java
class 子类名 extends 父类名 implenments 接口名
{...
}
```

`A a = new B(); `

结果a是一个A类的实例，只能访问A中的方法。那么又和`A a = new A();`有什么区别呢？

class B extends A 继承过后通常会定义一些父类没有的成员或者方法。

`A a = new B();`这样是可以的，上传。a是一个父类对象的实例，因而不能访问子类定义的新成员或方法。

------

假如这样定义：

```java
class A{
    int i;
    void f(){}
}
class B extends A{
    int j;
    void f(){}//重写
    void g(){}
}
```

然后：`B b = new B();`
b就是子类对象的实例，不仅能够访问自己的属性和方法，也能够访问父类的属性和方法。诸如b.i,b.j,b.f(),b.g()都是合法的。此时 b.f()是访问的B中的f()

`A a = new B();`
a虽然是用的B的构造函数，但经过upcast，成为父类对象的实例，不能访问子类的属性和方法。a.i,a.f()是合法的，而a.j,a.g()非 法。此时访问a.f()是访问B中的f()

------

A a = new B(); 这条语句，实际上有三个过程：
(1) A a;将a声明为父类对象，只是一个引用，未分配空间
(2) B temp = new B();通过B类的构造函数建立了一个B类对象的实例，也就是初始化
(3) a = (A)temp;将子类对象temp转换未父类对象并赋给a，这就是上传(upcast)，是安全的。
经过以上3个过程，a就彻底成为了一个A类的实例。
子类往往比父类有更多的属性和方法，上传只是舍弃，是安全的；而下传(downcast)有时会增加，通常是不安全的。

------

a.f()对应的应该是B类的方法f();调用构造函数建立实例过后，对应方法的入口已经确定了。
如此以来，a虽被上传为A类，但其中重写的方法f()仍然是B的方法f()。也就是说，每个对象知道自己应该调用哪个方法。
`A a1 = new B()；`
`A a2 = new C();`
a1,a2两个虽然都是A类对象，但各自的f()不同。这正是的多态性的体现。
这类问题在《Java编程思想》上都讲的很清楚.

**implements一般是实现接口。extends 是继承类。**

 接口一般是只有方法声明没有定义的，那么java特别指出实现接口是有道理的，因为继承就有感觉是父类已经实现了方法，而接口恰恰是没有实现自己的方法，仅仅有声明，也就是一个方法头没有方法体。因此你可以理解成接口是子类实现其方法声明而不是继承其方法。但是一般类的方法可以有方法体，那么叫继承比较合理。引入包可以使用里面非接口的一切实现的类。那么是不是实现接口，这个你自己决定，如果想用到那么你不是实现，是不能调用这个接口的，因为接口就是个规范，是个没方法体的方法声明集合。我来举个例子吧：接口可以比作协议，比如我说一个协议是“杀人”那么这个接口你可以用 砍刀去实现，至于怎么杀砍刀可以去实现，当然你也可以用抢来实现杀人接口，但是你不能用杀人接口去杀人，因为杀人接口只不过是个功能说明，是个协议，具体怎么干，还要看他的实现类。那么一个包里面如果有接口，你可以不实现。这个不影响你使用其他类。

implements是一个类实现一个接口用的关键字，他是用来实现接口中定义的抽象方法。比如：people是一个接口，他里面有say这个方法。public interface people(){ public say();}但是接口没有方法体。只能通过一个具体的类去实现其中的方法体。比如chinese这个类，就实现了people这个接口。

```java
public class chinese implements people{ 
    public say() {
        System.out.println("你好！");
    }
}
```

 与extends不同，在java中implements表示子类实现父类，如类A实现类B写成 class A implements B{}

* extends， 可以实现父类，也可以调用父类初始化 this.parent()。而且会覆盖父类定义的变量或者函数。这样的好处是：架构师定义好接口，让工程师实现就可以了。整个项目开发效率和开发成本大大降;
* implements，实现父类，子类不可以覆盖父类的方法或者变量。即使子类定义与父类相同的变量或者函数，也会被父类取代掉。

这两种实现的具体使用，是要看项目的实际情况，需要实现，不可以修改implements，只定义接口需要具体实现，或者可以被修改扩展性好，用extends。 
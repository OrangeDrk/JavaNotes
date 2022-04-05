[TOC]

# Super

> 1. super：指代当前对象的超类对象
> 2. super的用法：
>    - super.成员变量------访问超类的成员变量
>    - super.方法名()------调用超类的方法
>    - ==super()------调用超类的构造方法==
> 3. Java规定：构造子类之前必须调用构造父类
>   - 子类的构造方法中必须通过super关键字调用父类的构造方法，这样可以==妥善的初始化继承自父类的成员变量==
>   - 在子类的构造方法中若自己不调用超类的构造方法------则默认`super()`调用父类的无参构造方法
>   - ==在子类的构造方法中若调用了父类的构造方法------则不在默认==
>   - `super()`调用超类构造，==必须位于子类构造的第一行==（this也必须放在第一行，所以this和super不能同时使用）
> 4. this和super的区别
>    - this：指代当前对象
>    - super：指代继承的父类

```java
//Person类（父类）
package day04;

public class Person {
	String name;
	int age;
	Person(){
		
	}
	
	Person(String name,int age){
		this.name = name;
		this.age = age;
	}
	
	void eat() {System.out.println("吃饭");}
	void sleep() {System.out.println("睡觉");}
}


//Student类（子类）
package day04;

public class Student extends Person{
	int studentID;
	String sex;
	
	Student(){
		super("Oliver",25);
	}
	
	Student(int studentID,String sex){
		this.studentID = studentID;
		this.sex = sex;
	}
	
	void study(){System.out.println("学习");}
	void hitTeacher() {System.out.println("打老师");}

	@Override
	public String toString() {
		return "Student [studentID=" + studentID + ", sex=" + sex + ", name=" + name + ", age=" + age + "]";
	}
	
}



//Demo类（main方法）
package day04;

public class Demo {
	public static void main(String[] args) {
		Student s = new Student();
		System.out.println(s);
		Boo o = new Boo();
	}
	
}
class Aoo{
	String str;
	Aoo(){
		System.out.println("chaolei");
	}
}
class Boo extends Aoo{
	Boo(){
		super();
		System.out.println("peisheng");
	}
}
```



# 向上造型（父类指向子类）

1. 超类型的引用指向派生类的对象------目的：代码复用（抽象类和接口的地方用的较多）

2. 能点出来什么，看引用类型

3. 使用向上造型有两种情况：

   - 代码复用

   - 当类（抽象类）或者接口定义一个规则时，必须要用向上造型

     ```java
     public class UploadDemo {
     	public static void main(String[] args) {
     		//父类  指向  子类对象
     		A a = new B();//向上造型
     		a.a = 1;
     		a.show();//能点出来什么，看引用类型
     		//a.say();//编译错误，能点出来什么看前面的引用
     		//a.b = 1;//编译错误，能点出来什么看前面的引用
     	}
     }

     class A{
     	int a;
     	void show() {}
     }

     class B extends A{
     	int b;
     	void say() {}
     }
     ```



> 向上造型的一个用处

```java
package day04;

public class OverrideDemo01 {
        public static void main(String[] args) {
                Goo goo = new Goo();
                Eoo o = new Foo();
                goo.test(o);
        }
}

class Goo{
        void test(Eoo o) {
                System.out.println("超类型参数");
                o.show();
        }
        
        void test(Foo o) {
                System.out.println("派生类型参数");
                o.show();
        }
}

class Eoo{
        void show() {
                System.out.println("超类show");
        }
}

class Foo extends Eoo{
        void show(){
                System.out.println("派生类show");
        }
}

```



> 向上造型后可以传父类引用类型参数、子类重写父类方法后，向上造型点（.）重写方法调用子类重写的方法

```java
package day04;

public class OverrideDemo {
        /**
         * Object类是所有类的父类
         * 当一个类没有继承关系时，默认情况下都继承Object类，就算不写，也默认继承,Object类是所有类的父类
         * @param args
         */

        public static void main(String[] args) {
                Xoo xoo = new Yoo();
                xoo.show();
        }
        /**
         * 当参数设为父类时，我们可以接收父类的所有继承的对象，
         * 	      这样我们只需要定义一个方法即可，就可以接收向上造型所指向的所有子类对象
         */
        public static void println(Xoo o) {
                
        }

}

class Xoo{
        void show() {
                System.out.println("父类方法");
                }
}

class Yoo extends Xoo{
        void show() {
                System.out.println("Yoo子类的方法show()");
        }
        
        void say() {
                System.out.println("Yoo子类的方法say()");
        }
}


```



# 方法的重写

> 重新写，覆盖------觉得父类方法不够好，满足不了子类的需求。足够好就不用重写

​	发生在父子类中，子类可以重写（覆盖）继承自父类的方法，即方法名相同，参数列表相同，方法体不同，即方法的实现不同

> @Override：重写

该注解声明在方法上，标明该方法必须要重写父类的方法



## 重写方法被调用时，看对象的类型

> 当子类对象的重写方法被调用时（无论通过子类的引用调用还是通过父类的引用调用），运行的是子类的重写后的版本



## 重写须遵循”两同两小一大“

- 两同

  - 方法名、参数列表相同
  - 父类中的返回值是：基本数据类型、final最终类、void时，子类重写时返回值类型必须相同

- 两小

  - 子类方法的返回值类型小于或等于父类方法（==子类重写的方法的返回值类型时父类返回值类型的子类或者本身==）
    - 引用类型时，小于或等于（父类>子类）
  - 子类方法抛出的异常小于或等于父类的（==和父类抛出相同的异常，或者抛出父类异常的子类异常==）
  
- 一大

  - 子类方法的访问权限大于或等于父类方法的

    ```java
        interface b{
        	void cut();
        }
        abstract class c{
        	abstract void cut();
        }
    
        class a extends c{
        	//继承抽象类，方法修饰符是abstract，重写大于或等于abstract
    
        void cut() {}
        }
        class d implements b{
        	//实现接口，接口方法默认为：public abstract，重写大于或等于public abstract
    
        	public void cut() {}
        }
    
    ```

```java
	package day04;
	
	public class OverrideAndOverload {
	        
	}
	class Coo{
	        void show() {};
	        double test() {return 0.0;}
	        Doo say() {return null;}
	        Coo sayHi() {return null;}
	}
	
	class Doo extends Coo{
	        //int show(){return 1;}//编译错误，void类型必须相同
	        //int test(){return 0;}//编译错误，基本类型必须相同
	        //Coo say(){return null;}//编译错误，引用类型必须小于或等于
	        //Doo sayHi() {return null;}//编译正确，引用类型必须小于或等于
	        
	        void show() {};
	        double test() {return 0.0;}
	        Doo say() {return null;}
	        Doo sayHi() {return null;}
	}

```



## 重写与重载的区别（常见面试题）

1. 重写
   - ==发生在父子类中，方法名相同，参数列表相同，方法体不同==
   - 遵循“运行期绑定”，看对象的类型调用方法
2. 重载
   - ==发生在一个类中，方法名相同，参数列表不同，方法体不同==
   - 遵循“编译器绑定”，看参数/引用类型来绑定
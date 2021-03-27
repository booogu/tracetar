---
layout:     post
comments: true
title:      JVM方法调用原理
subtitle:   解析、静态/动态分派
date:       2021-02-24 22:23:00
author:     "Booogu"
header-style: text
catalog: true
tags:
    - JVM
    - 读书笔记
    - 基础
---

## 方法调用的含义
方法调用仅仅指的是，确定被调用方法的版本（即确定调用哪个方法），而不等同于方法中的代码执行。方法调用共分为解析、分派两类：
### 解析
对应类加载过程中的”解析“阶段，指的是是把字节码中的部分符号引用转化为直接引用，这一步骤的前提是：方法在程序真正运行之前就有一个可确定的调用版本，并且此调用版本在运行期是不可变的。这意味着：方法的调用目标，在代码写好，编译器进行编译的那一刻就已经确定下来，这种方法的调用就是解析。<br>

而满足”编译期可知，运行期不变“这个要求的方法，主要有静态方法和私有方法两大类。前者与类型直接关联，后者在外部不可被访问。这类方法各自的特点都决定了：它们都不可能通过继承或别的方式重写出其他版本。<br>

Java中调用不同的方法，字节码指令集里设计了不同的指令。在Java虚拟机支持以下5种：
- Invokestatic 调用静态方法
- Invokespecial 调用实例构造器（<init>()方法）、私有方法和父类中的方法
- Invokevirtual 调用所有的虚方法
- Invokeinterface 调用接口方法，会在运行时再确定一个实现该接口的对象

Invokedynamic 先在运行时动态解析出调用点限定符所引用的方法，然后再执行该方法。前面4条调用指令，分派逻辑都固化在Java虚拟机内部，而invokedynamic指令的分派逻辑是由用户设定的引导方法来决定的。

**注意**：invokestatic和invokespecial指令调用的方法，都可以在解析阶段中确定唯一的调用版本，共有：静态方法、私有方法、实例构造器、父类方法4种，再加上被final修饰的方法（遗留原因，使用invokevirtual指令调用），这五类称为非虚方法，其余的方法，都是虚方法


### 分派
这一词语本身就具有动态性，可以分为静态单分派，静态多分派，动态单分派，动态多分派

#### 静态分派
静态分派的典型应用为方法重载<br>
方法重载中， 如：Human man = new Man() / new Woman(); 左侧的Human是变量"man"的静态类型/外观类型，后面的Man/Woman被称为变量的实际类型/运行时类型，其中静态类型是编译器可知的，而实际类型的变化结果是运行期才可确定的，因此其实际类型，再编译期间，是”薛定谔的人“，只有在运行期才可知，因此，JVM（准确说是编译器，JVM还未参与）在重载情况下，是根据参数的静态类型而非实际类型来作为判定依据的

#### 动态分派
典型应用为方法重写，我们把这种在运行期根据实际类型确定方法执行版本的分派过程称为动态分派<br>
方法重写中，调用的JVM指令为invokevirtual，这里指令的参数指向的符号引用，仍然是参数静态类型对应的方法的符号引用，但是invokevirtual指令执行时，却不再时简单把常量池中的符号引用”解析“到直接引用（直接调静态类型对应的方法），而是要执行以下几步：
- 找到操作数栈顶的第一个元素指向的对象的”实际类型“，记作C
- 如果在类型C中找到与常量中的描述符和简单名称都相符（注意：这一步就是在找父类中被重写的方法，很关键！）的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找过程结束；不通过则返回java.lang.IllegalAccessError异常。
- 否则，按照继承关系从下往上依次对C的各个父类进行第二步的搜索和验证过程。
- 如果自始至终没有找到合适的方法，则抛出"java.lang.AbstractMethodError"异常

> 正是因为invokevirtual指令执行的第一步就是在运行期确定接收者的实际类型，所以调用中的invokevirtual指令并不是把常量池中方法的符号引用解析到直接引用上就结束了，还会根据方法接收者的实际类型来选择方法版本，这个过程，就是Java语言中方法重写的本质！


**注意**：字段不具有多态性，字段不可能和方法一样是虚的，invokevirtual只针对方法！
以下代码辅助理解动态分配invokevirtual过程，并证明字段不具有多态性：

````java
public class FieldHasNoPolymorphic {

    static class Father {
        public int money = 1;

        Father() {
            this.money = 3;
            showMeTheMoney();
        }

        public void showMeTheMoney() {
            System.out.println("I'm father, i have $" + money);
        }
    }

    static class Child extends Father {
        public int money = 4;

        Child() {
            this.money = 2;
            showMeTheMoney();
        }

        @Override
        public void showMeTheMoney() {
            System.out.println("I'm child, i have $" + money);
        }
    }

    public static void main(String[] args) {
        Father father = new Child();
        System.out.println("This gay has $ " + father.money);
    }

}
````

以上代码的输出为：
I'm child, i have $0<br>
I'm child, i have $2<br>
This gay has $ 3

**分析**
- 创建子类实例时，隐式调用父类构造器，父类构造器中调用showMeTheMoney的静态类型是Father，但实际类型是Child，所以按照invokevirtual的逻辑，会通过动态分配，调用实际类型的方法，即子类方法，而这时子类还没有实例化，所以子类money的零值是0，首先输出0，
- 第二步，执行子类实例创建，输出的是子类的money值2
- 最后一步，这里拿到的是，静态类型为Father的字段，那么根据字段不具有多态，这里的字段就是父类本身的字段，所以输出的是3.

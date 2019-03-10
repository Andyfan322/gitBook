# 命令模式

>命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。

#### 介绍

* 意图：将一个请求封装成一个对象，从而使您可以用不同的请求对客户进行参数化。
* 主要解决：在软件系统中，行为请求者与行为实现者通常是一种紧耦合的关系，但某些场合，比如需要对行为进行记录、撤销或重做、事务等处理时，这种无法抵御变化的紧耦合的设计就不太合适。
* 关键代码：定义三个角色：**1、Receiver 真正的命令执行对象 2、Command 命令3、Invoker 命令调用协调者** 
* 流程：调用者→接受者→命令。
* 优点： 1、降低了系统耦合度。 2、新的命令可以很容易添加到系统中去。
* 缺点：使用命令模式可能会导致某些系统有过多的具体命令类。

#### 例子

```java 
//抽象命令
public abstract class Command {

    public abstract void execute();
    
}
//具体命令1
public class ConcreteCommand1 extends Command {
    private Receiver receiver;

    public ConcreteCommand1(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.doSomething();
    }
}
// 具体命令2
public class ConcreteCommand2 extends Command {
    private Receiver receiver;

    public ConcreteCommand2(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.doSomething();
    }
}
// 具体命令执行者抽象类
public abstract class Receiver {

    public abstract void doSomething();
}
//具体命令执行者1
public class Receiver1 extends  Receiver {
    @Override
    public void doSomething() {

    }
}
//具体命令执行者2
public class Receiver2 extends  Receiver {
    @Override
    public void doSomething() {
        
    }
}
//命令调用的协调者
public class Invoker {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void action() {
        command.execute();
    }
}
//test方法
public class Main {
    public static void main(String[] args){
        Receiver receiver = new Receiver1();
        Command command = new ConcreteCommand1(receiver);

        Invoker invoker = new Invoker();
        invoker.setCommand(command);
        invoker.action();
    }
}

```

#### 核心思路

* 执行者抽象
* 命令抽象（里面包含命令的执行者）
* 事务协调者（持有命令对象）


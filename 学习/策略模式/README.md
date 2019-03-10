# 策略模式

> 在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。
在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。

#### 介绍
* 意图：定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。
* 主要解决：在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护。
* 何时使用：一个系统有许多许多类，而区分它们的只是他们直接的行为。
* 如何解决：将这些算法封装成一个一个的类，任意地替换。
* 关键代码：实现同一个接口。
* 应用实例： 1、诸葛亮的锦囊妙计，每一个锦囊就是一个策略。 2、旅行的出游方式，选择骑自行车、坐汽车，每一种旅行方式都是一个策略。 3、JAVA AWT 中的 LayoutManager。
* 优点： 1、算法可以自由切换。 2、避免使用多重条件判断。 3、扩展性良好。
* 缺点： 1、策略类会增多。 2、所有策略类都需要对外暴露。

```java
public interface GoToBeijing {

    /**
     * 去北京
     */
    void toBeijing();

}
public class Bus implements GoToBeijing {

    /**
     * 坐汽车
     */
    @Override
    public void toBeijing() {
        System.out.println("go to beijing byBus");
    }
}
public class Fly implements GoToBeijing {

    /**
     * 坐飞机
     */
    @Override
    public void toBeijing() {
        System.out.println("go to beijing take a plane");
    }
}
public class StrategyContext {

    private GoToBeijing toBeijing;

    public StrategyContext(GoToBeijing toBeijing) {
        this.toBeijing = toBeijing;
    }

    public void toBeijing() {
        toBeijing.toBeijing();
    }
}

public class Main {

    public static void main(String[] args) {

        StrategyContext context = new StrategyContext(new Bus());
        context.toBeijing();
        StrategyContext context2 = new StrategyContext(new Fly());
        context2.toBeijing();
    }

}
```

#### 核心思路
* 事件的抽象
* 实现方法分类
* 调度实现

> 与命令模式很相像，我觉得他们之间的区别在于解决问题的角度不一样，命令模式侧重于请求和执行2者解耦，策略强调的是事务多种解决方案或者途径。
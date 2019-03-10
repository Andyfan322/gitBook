# 建造者模式

> 建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
一个 Builder 类会一步一步构造最终的对象。该 Builder 类是独立于其他对象的。

#####  介绍

* 意图：将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。
* 主要解决：主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定。
* 何时使用：一些基本部件不会变，而其组合经常变化的时候。
* 如何解决：将变与不变分离开。
* 关键代码：建造者：创建和提供实例，导演：管理建造出来的实例的依赖关系。
* 应用实例： 1、去肯德基，汉堡、可乐、薯条、炸鸡翅等是不变的，而其组合是经常变化的，生成出所谓的"套餐"。 2、JAVA 中的 StringBuilder。
* 优点： 1、建造者独立，易扩展。 2、便于控制细节风险。
* 缺点： 1、产品必须有共同点，范围有限制。 2、如内部变化复杂，会有很多的建造类。


#### 例子

```java
public abstract class AbstractBuilder {

    abstract void buildPart1();

    abstract void buildPart2();

    abstract void buildPart3();
}

public class BuilderImpl extends AbstractBuilder {
    @Override
    void buildPart1() {
        System.out.println("构建对象第一步");
    }

    @Override
    void buildPart2() {
        System.out.println("构建对象第二步");
    }

    @Override
    void buildPart3() {
        System.out.println("构建对象第三步");
    }
}

public class Director {

    private AbstractBuilder builder;

    private Integer condition;

    public Director(AbstractBuilder builder, Integer condition) {
        this.builder = builder;
        this.condition = condition;
    }

    public void build(AbstractBuilder builder) {
        switch (condition) {
            case 1:
                builder.buildPart1();
                break;
            case 2:
                builder.buildPart2();
                break;
            case 3:
                builder.buildPart3();
                break;
            default:
                builder.buildPart1();
        }
    }
}
```

#### 核定思路

 * 对象构建需要若干步骤
 * 构建抽象类，抽象每一步
 * 一个协调者负责协调构建的每一步
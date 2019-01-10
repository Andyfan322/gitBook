# String

>String对于任何一门语言的重要性，不必言语，在Java里面String被设计成了final类型，底层使用char[]数组来实现，并且它还重写了equals方法。


* 整体

``` java 
public final class String implements java.io.Serializable, Comparable<java.lang.String>, CharSequence {

    private final char value[];//存储具体的值的数组

    private int hash; // hash值

    private static final long serialVersionUID = -6849794470754667710L;

    private static final ObjectStreamField[] serialPersistentFields =
            new ObjectStreamField[0];

  //构造方法1，底层还是用了char[]承载
    public String() {
        this.value = new char[0];
    }

    public String(java.lang.String original) {
        this.value = original.value;
        this.hash = original.hash;
    }
   //构造方法2
    public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }

  //构造方法3
    public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
···
```

>这里列出了String全部filed，他们都是final类型的，和几个常用的构造方法，可以看到String底层使用char[]来承载具体的数值。

* 常见的方法

```
   public char charAt(int index) {
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        return value[index];
    }
    
    public String substring(int beginIndex, int endIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        if (endIndex > value.length) {
            throw new StringIndexOutOfBoundsException(endIndex);
        }
        int subLen = endIndex - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
    }
    
    public String concat(String str) {
        int otherLen = str.length();
        if (otherLen == 0) {
            return this;
        }
        int len = value.length;
        char buf[] = Arrays.copyOf(value, len + otherLen);
        str.getChars(buf, len);
        return new String(buf, true);
    }        
```    
> String 是immutable（不可变的），这写方法的操作都是使用了新的char[],原始的字符串是不可变的。即String一旦创建，对它的任何操作都是不会原值，会再生成新的对象。


* StringBuilder 、StringBUffer

    * 线程安全否？		
    
    String是immutable，因此必然是线程安全的，而StringBuilder和StringBuffer是可变的，故存在线程安全问题
    
    * StringBUilder

    ```java
    public final class StringBuilder extends AbstractStringBuilder implements java.io.Serializable, CharSequence
{

      public StringBuilder() {
        super(16);
    }

    public StringBuilder(int capacity) {
        super(capacity);
    }

 
    public StringBuilder(String str) {
        super(str.length() + 16);
        append(str);
    }

    public StringBuilder(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }

    @Override
    public StringBuilder append(Object obj) {
        return append(String.valueOf(obj));
    }
 } 
abstract class AbstractStringBuilder implements Appendable, CharSequence {
//底层也是char[]来装实际的值
    char[] value;

    int count;

    AbstractStringBuilder() {
    }

    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
// 真正的append方法，可以扩容
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
}
    ```
>可以看到StringBuilder底层也是数组，默认char[]大小是16，但是它可以扩容，执行append操作实现上调用的都是父类的appnd方法，且在其内部没看到任何线程安全的保护措施，因此使用它需要注意线程安全问题。


* StringBuffer

```java
public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
{
    private transient char[] toStringCache;

    static final long serialVersionUID = 3388685877147921107L;

//底层也是char[]来装实际的值
    public StringBuffer() {
        super(16);
    }

    public StringBuffer(int capacity) {
        super(capacity);
    }

    public StringBuffer(String str) {
        super(str.length() + 16);
        append(str);
    }
   public synchronized StringBuffer append(StringBuffer sb) {
        toStringCache = null;
        super.append(sb);
        return this;
    }
  }
abstract class AbstractStringBuilder implements Appendable, CharSequence {
   
    char[] value;

    int count;

    AbstractStringBuilder() {
    }
    
    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
 }   
```    
>可以看到StringBuffer底层也是数组，默认char[]大小是16，但是它可以扩容，执行append操作实现上调用的都是父类的appnd方法，且在其内部方法都用synchronized修饰，是线程安全的。
    
    
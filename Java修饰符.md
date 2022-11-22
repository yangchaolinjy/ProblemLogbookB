| 修饰符 | 当前类 | 同一包内 | 子孙类(同一包) | 子孙类(不同包) | 其他包 |
| --- | --- | --- | --- | --- | --- |
| public | Y | Y | Y | Y | Y |
| protected | Y | Y | Y | Y/N（[说明](https://www.runoob.com/java/java-modifier-types.html#protected-desc)<br />） | N |
| default | Y | Y | Y | N | N |
| private | Y | N | N | N | N |

<a name="x1FZz"></a>
### protected
protected 需要从以下两个点来分析说明：

- **子类与基类在同一包中**：被声明为 protected 的变量、方法和构造器能被同一个包中的任何其他类访问；
- **子类与基类不在同一包中**：那么在子类中，子类实例可以访问其从基类继承而来的 protected 方法，而不能访问基类实例的protected方法。
<a name="G3uTY"></a>
### transient 

- 序列化的对象包含被 transient 修饰的实例变量时，java 虚拟机(JVM)跳过该特定的变量。
- 该修饰符包含在定义变量的语句中，用来预处理类和变量的数据类型。
<a name="OHrgl"></a>
### volatile 

- olatile 修饰的成员变量在每次被线程访问时，都强制从共享内存中重新读取该成员变量的值。而且，当成员变量发生变化时，会强制线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。一个 volatile 对象引用可能是 null。
```java
public class MyRunnable implements Runnable
{
    private volatile boolean active;
    public void run()
    {
        active = true;
        while (active) // 第一行
        {
            // 代码
        }
    }
    public void stop()
    {
        active = false; // 第二行
    }
}
```

- 通常情况下，在一个线程调用 run() 方法（在 Runnable 开启的线程），在另一个线程调用 stop() 方法。 如果 **_第一行_** 中缓冲区的 active 值被使用，那么在 **_第二行_** 的 active 值为 false 时循环不会停止。
- 但是以上代码中我们使用了 volatile 修饰 active，所以该循环会停止。

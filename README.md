# Java_Nodes
Java-消除过期对象的引用
众所周知，在Java语言中，有垃圾回收机制，这让程序员脱离了手工管理内存（如C、C++）的痛苦，**可这并不代表一个对象被用完之后就会被回收**。先贴上一个简单栈的例子。
```
/**
 * @author 田泽鑫
 * @date 2019/11/8
 */
public class MyStack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public MyStack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e){
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop(){
        if (size == 0){
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size){
            elements = Arrays.copyOf(elements,2*size + 1);
        }
    }
}
```

   这段代码并没有明显的错误，测试也都能通过。但是实际上这个程序隐藏了一个问题——**"内存泄露"**，当垃圾回收器活动的增加，或者由于内存占用不断增加，程序的性能的降低就会表现出来。在极端的情况下，内存泄露会导致**磁盘交换**，程序崩溃，但这些情况比较少见。
   如果一个栈先是增长（push），然后又压缩弹出(pop)，那么，从栈中弹出来的对象将不会被当做垃圾回收，即使使用栈的程序不再引用这些对象，他们也不会被回收。因为栈内部维护着对这些对象的**过期引用**，即永远也不会再被解除的引用。请看简单的测试

```/**
 * @author 田泽鑫
 * @date 2019/11/8
 */
public class test {
    public static void main(String[] args) {
        MyStack myStack = new MyStack();
        Person person1 = new Person("小明1");
        Person person2 = new Person("小明2");
        Person person3 = new Person("小明3");
        myStack.push(person1);
        myStack.push(person2);
        myStack.push(person3);
        myStack.pop();
  	System.out.println(person3);
    }
}
```
debug模式可以看到，还没有pop的时候，是如图所示
![image.png](http://49.233.148.68:8090/upload/2019/11/image-c5c38edf116f4fe2b6b6894ca4de9530.png)
继续执行断点，执行pop语句之后，可以看到栈内只有两个元素
![image.png](http://49.233.148.68:8090/upload/2019/11/image-79a189bab1cc4a368037d146d8438e58.png)
但是element[2]这个引用还是还可以指向真实的对象Person@460，这个引用并未被回收。
![image.png](http://49.233.148.68:8090/upload/2019/11/image-511f7d01be2a4bb6b6ff071c2833230b.png)
在这个例子中，element[2]这个对象引用指向对象Person@460，element[2]即是上面提到的过期引用，element[2]是**过期引用**。
**这里要提一下person3这个引用，person3并不是过期引用**
![image.png](http://49.233.148.68:8090/upload/2019/11/image-ca76686062564bc298e3118a8bc2eb36.png)，
person3这个对象引用依然可以指向Person@460，它是存在于test这个测试程序，跟MyStack类没有关系，而且person3这个对象引用是一个强引用，在程序未结束之前是不会被回收的。
要解决上述代码的问题，只需要在对象引用过期的时候，清空这些引用即可，修改如下。
```
public Object pop(){
        if (size == 0){
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

```
清空过期引用的另一个好处在于，如果它们以后又被错误地解除引用，程序就会立即抛出NullPointerException异常。

- 参考资料《Effective Java》

关于内存泄露还有很多高深的东西，我只是简单的了解这些概念。
2019年11月9日16:36:38

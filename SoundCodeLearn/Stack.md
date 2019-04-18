### Stack

> Stack是java.lang.util包下的一个类，继承了集合Vector，具有先进后出的特性，可以借助数组和链表实现这样的数据结构。之前一直没有机会接触真正使用过栈，最近做了leetcode中《有效的括号》那道题，才真正的用到了它。PS：一开始没用栈做那道题导致的后果就是花费了几十分钟、写了上百行代码。
  
### 方法简介
|  方法名| 说明 |
|--|--|
|  push()| 将元素推入栈 |
|  pop()| 弹出栈顶端的元素 |
|  peek()|返回栈顶端的元素  |
|  empty()| 判断栈是否为空 |
|  search()| 返回最靠近顶端的目标元素到顶端的距离 |
### 源码阅读
> Stack继承了Vector集合类，所以其具有Vector的一些特性，存取元素等等功能也都是用Vector取完成的，Stack类做的活很少，源码也自然很少。所以最主要的还是了解栈的概念。

 1. push()
```java
	public E push(E item) {
		//调用父类Vector的方法
        addElement(item);
        return item;
    }
	//Vector类添加元素方法
	public synchronized void addElement(E obj) {
        modCount++;
        //扩容实现
        ensureCapacityHelper(elementCount + 1);
        //把元素追加到末尾位置
        elementData[elementCount++] = obj;
    }
```

 2. pop()

```java
	public synchronized E pop() {
        E       obj;
        //获取数组长度
        int     len = size();

		//通过peek()方法取出末尾元素
        obj = peek();
        //调用父类Vector的方法，删除末尾元素
        removeElementAt(len - 1);

        return obj;
    }
```

 3. peek()

```java
	//获取末尾元素
	public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }
```

 4. empty()

```java
	public boolean empty() {
        return size() == 0;
    }
```

 5. search(Object o)

```java
	public synchronized int search(Object o) {
		//获取元素o的位置
        int i = lastIndexOf(o);

        if (i >= 0) {
        	//返回数组个数-o的位置，即获得元素靠近顶端的距离
            return size() - i;
        }
        return -1;
    }
```

> Stack类的代码就这么多了，一百多行的代码(还有一大半的注释)，主要的实现都是在Vector类内，而其实Vector和ArrayList功能基本一样，只不过Vector在很多方法上加了厉害的sychronize罢了，所以这三个类的学习主要看下ArrayList的实现，然后重点是要理解各自的用途。

## 一、简介
&emsp;&emsp; ArrayList是基于数组实现的一个动态数组，其容量能自动增长。ArrayList不是线程安全的，多线程环境下考虑Collections.synchronizedList(List l)函数返回一个线程安全的ArrayList类，也可以使用concurrent并发包下的CopyOnWriteArrayList类。
## 二、源码理解
### 2.1 类继承关系
```
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
ArrayList继承AbstractList抽象父类，实现了List接口（规定了List的操作规范）、RandomAccess（可随机访问）、Cloneable（可拷贝）、Serializable（可序列化）
### 2.2 类属性

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
	//序列化版本号
    private static final long serialVersionUID = 8683452581122892189L;
    //默认数组容量
    private static final int DEFAULT_CAPACITY = 10;
    //空实例，默认为{}，通过带参数构造器创建实例时使用
    static final Object[] EMPTY_ELEMENTDATA = {};
    //缺省空实例，采用ArrayList默认大小10创建的空实例
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    //用来存储真正的数据数组
    transient Object[] elementData; 
    //数组元素数量
    private int size;
    //最大数组容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
}
```

### 2.3 构造函数

 1. 有参构造器(初始化数组容量)
	```java
	 public ArrayList(int initialCapacity) {
	 		//如果initialCapacity大于0则根据指定的大小创造一个数组
	        if (initialCapacity > 0) {
	            this.elementData = new Object[initialCapacity];
	        //如果initialCapacity等于0则将默认的空列表EMPTY_ELEMENTDATA赋值给elementData
	        } else if (initialCapacity == 0) {
	            this.elementData = EMPTY_ELEMENTDATA;
	       //小于0则抛出参数异常
	        } else {
	            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
	        }
	    }
	```

 2. 无参构造器

	```java
	 public ArrayList() {
	 		//将DEFAULTCAPACITY_EMPTY_ELEMENTDATA空数组赋值给elementData
	        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
	    }
	```

 3.  有参构造器(根据给定的集合)

		```java
		public ArrayList(Collection<? extends E> c) {
	        elementData = c.toArray();
	        if ((size = elementData.length) != 0) {
	           //如果elementData长度不等于0，并且不是Object类型数组时，则调用Arrays.copyOf复制一个Object类型数组，并复制给elementData
	            if (elementData.getClass() != Object[].class)
	                elementData = Arrays.copyOf(elementData, size, Object[].class);
	        } else {
	            //如果数组大小等于0，则将EMPTY_ELEMENTDATA空数组赋值给elementData 
	            this.elementData = EMPTY_ELEMENTDATA;
	        }
	    }
		```

### 2.4 核心函数

 - add(E e):添加元素
	
	```java
	public boolean add(E e) {
			//调用ensureCapacityInternal函数，确定数组的大小，size+1表示添加当前元素后数组的容量大小
	        ensureCapacityInternal(size + 1); 
	        //将数组size位置的元素设置为e
	        elementData[size++] = e;
	        return true;
	    }
	```
 
	
	```java
	private void ensureCapacityInternal(int minCapacity) {
		//如果数组是默认的空数组，则将minCapacity与默认数组的容量大小比较，取最大的赋值给minCapacity(需要的数组容量值)
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
		//确定是否需要数组扩容
        ensureExplicitCapacity(minCapacity);
    }
	```
	```java
	private void ensureExplicitCapacity(int minCapacity) {
	       //记录集合的修改次数
	        modCount++;
			//如果需要的数组容量大小大于当前数组大小，则调用grow()方法进行扩容
	        if (minCapacity - elementData.length > 0)
	            grow(minCapacity);
	    }
	```
	```java
	//这个方法是ArrayList这个类add方法的最后一步，主要做的就是确定size然后进行数组的copy与赋值
	//至于真正的copy，是由Arrays.copyOf调用System.arraycopy，最后完成copy这一步是个native方法去做的
	private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        //newCapacity当前数组容量，取原数组的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果暂定数组容量不满足指定容量minCapacity，则将minCapacity赋值给newCapacity
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果newCapacity大于最大容量，则调用hugeCapacity()方法确定最终值
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        //调用Arrays.copyOf(原数组，数组大小)，将数组copy形成新的数组重新复制，从而完成数组的扩容
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
	```
	```java
	//超容量数组大小判断
	private static int hugeCapacity(int minCapacity) {
	        if (minCapacity < 0) // overflow
	            throw new OutOfMemoryError();
	        //如果minCapacity 大于MAX_ARRAY_SIZE则取Integer的最大值，如果小于则取MAX_ARRAY_SIZE
	        return (minCapacity > MAX_ARRAY_SIZE) ?
	            Integer.MAX_VALUE :
	            MAX_ARRAY_SIZE;
	    }
	```
	通过add方法的阅读可以发现，几个重要的点
	
	 - ArrayList本质上是个数组，数据存储是由数组来完成，ArrayList只不过是进行了封装、扩展功能
	 - 数组容量最大为Integer的最大值
	 - 添加元素内部实现，其实是建立的新的数组，所以再创建ArrayList时最好有个预估容量大小，不然会造成频繁的扩容

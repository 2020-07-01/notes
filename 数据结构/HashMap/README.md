

https://blog.csdn.net/qq_41357573/article/details/88806145?spm=1001.2014.3001.5501

## 关于HashCode()和equals()

```java
//HashMap源码
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

> Object的equals()方法比较的是对象的引用值，如果需要比较对象的属性，需要重写
>
> 不同的对象的hashcode是不同的

当HashMap的key是对象时，如果要比较对象里面的内容，则需要重写equals()方法和hashcode方法








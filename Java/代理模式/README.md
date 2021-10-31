

**思想**：代理模式通俗将就是将业务类中的某些业务方法交给代理类去执行，代理类在执行业务方法前后执行一个方法，对业务方法进行增强

**场景**：比如管理系统日志的收集，事务管理

代理模式有静态代理，动态代理，下面通过例子记录自己的理解

## 静态代理

场景：打印出请求参数

```java
//公共接口
public interface ParamsHandler {

    /**
     * 删除操作
     *
     * @param param
     */
    void delete(String param);

    /**
     * 更新操作
     *
     * @param param
     */
    void update(String param);
}

//业务类
public class BizService implements ParamsHandler {

    @Override
    public void delete(String param) {
        System.out.println("biz delete ......");
    }

    @Override
    public void update(String param) {
        System.out.println("biz update......");
    }
}

//业务代理类
public class BizProxy implements ParamsHandler {
    private BizService bizService;

    BizProxy(BizService bizService) {
        this.bizService = bizService;
    }

    @Override
    public void delete(String param) {
        System.out.println("打印参数......" + param);
        bizService.delete(param);
    }

    @Override
    public void update(String param) {
        System.out.println("打印参数......" + param);
        bizService.delete(param);
    }
}

public class Main {
    public static void main(String[] args) {

        BizService bizService = new BizService();
        BizProxy proxy = new BizProxy(bizService);
        proxy.delete("2");
        proxy.update("1");
    }
}
```

> 1.业务类和代理类实现同一个接口，代码冗余
>
> 2.硬编码方式，维护不方便
>
> 3.每个业务接口都是实现一个代理类



## jdk动态代理

```java
// 用户服务接口
public interface UserService {

    void addUser(String name);
}

public class UserServiceImpl implements UserService {

    @Override
    public void addUser(String name) {
        System.out.println("添加用户......" + name);
    }
}

//代理类
public class BizProxy implements InvocationHandler {

    //业务类
    Object target = null;

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("jdk动态代理开始......");
        Object o = method.invoke(target, args);
        System.out.println("jdk动态代理结束......");
        return o;
    }

    /**
     * 由目标对象获取代理对象
     *
     * @param targetObject
     * @return
     */
    public Object getProxyObject(Object targetObject) {
        this.target = targetObject;
        return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(), targetObject.getClass().getInterfaces(), this);
    }

    public static void main(String[] args) {
        BizProxy userProxy = new BizProxy();
        UserService userService = (UserService) userProxy.getProxyObject(new UserServiceImpl());
        userService.addUser("admin");
    }
}
```

> 1.多个业务类可以共用同一个代理类
>
> 2.业务类必须要实现一个接口

## cglib



```java
//代理类
public class UserProxy implements MethodInterceptor {
    
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        before(method.getName());
        Object result = methodProxy.invokeSuper(o, objects);
        after(method.getName());
        return null;
    }

    /**
     * 调用invoke方法之前执行
     */
    private void before(String methodName) {
        System.out.println("方法前执行" + methodName);
    }

    /**
     * 调用invoke方法之后执行
     */
    private void after(String methodName) {
        System.out.println("方法后执行" + methodName);
    }
}

//业务类
public class UserService {
    public void adduser(String name) {
        System.out.println("添加用户......" + name);
    }
}

public class Main {
    public static void main(String[] args) {
        /**
         * 通过cglib动态代理获取代理对象
         */
        Enhancer enhancer = new Enhancer();
        //设置目标类的字节码文件
        enhancer.setSuperclass(UserService.class);
        //设置回调函数
        enhancer.setCallback(new UserProxy());
        //创建代理类
        UserService userService = (UserService) enhancer.create();
        userService.adduser("admin");
    }
}
```

> 1.代理类不需要实现接口
>
> 2.通过字节码方式创建代理类的子类，对业务类进行增强
>
> 3.final 类型的方法不能被重写，final类不能被继承

## asm

> Java字节码处理框架，对class文件进行处理



## javassist

> Java编程助手，对字节码进行处理


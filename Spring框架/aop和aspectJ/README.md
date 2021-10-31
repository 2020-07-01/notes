## AOP

> 一种编程思想







## aspectJ

> 一种AOP实现框架





切面：@Aspect注解，包含切点和通知



连接点:上下文信息



强制使用cglib

> ```
> @EnableAspectJAutoProxy(proxyTargetClass = true)
> ```





spring 在启动过程中会为业务类创建代理类


> SpringBoot在启动过程中对Bean 进行自动注入



## spring，springmvc,springboot

Spring 是一个基于IOC和AOP的框架，使编码更加干净，bean的管理更加方便

Springmvc是spring的一个模块，是一个web框架，主要针对网站开发

SpringBoot是spring的扩展，约定大于配置，简化了spring的配置过程





1. @SpringBootAppclication
2. @EnableAutoConfiguration
3. @Import(AutoConfigurationImportSelector.class)



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
	//
}
```



```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
 	//   
}
```



## @Import

@Import是SpringBoot自动注入的核心

* 参数为普通类时，会自动注入到容器中

* **参数实现ImportSelector时，会进行批量注入bean**

* 参数实现ImportBeanDefinitionRegister时，支持手动注册bean 

  

```java
public interface DeferredImportSelector extends ImportSelector {
	//。。。。。。
}
```



```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
		ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

	 //省略代码
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```



```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    configurations = removeDuplicates(configurations);
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = getConfigurationClassFilter().filter(configurations);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}

```





## spring.factories

在启动过程中通过@Import(AutoConfigurationImportSelector.class)注解，找到spring.facatories

注入里面配置的bean



starter包主要用来管理依赖及其版本

autoconfigure 主要用来自动注入bean及逻辑代码实现





# 自定义@EnableXXX注解

> 重要


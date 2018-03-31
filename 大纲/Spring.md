# Spring

## 1. 为什么要使用Spring？

轻量级的容器框架没有侵入性

使用IoC容器更加容易组合对象直接间关系，面向接口编程，降低耦合
Aop可以更加容易的进行功能扩展，遵循ocp开发原则（开闭原则，一个程序应该是通过扩展来实现变化，而不是通过修改已有代码来实现变化）

创建对象默认是单例的，不需要再使用单例模式进行处理

## 2. 什么是IoC？

控制反转，有了IoC容器后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活。

## 3. 为什么不直接使用工厂模式。工厂模式也可以管理实例的初始化，为什么一定要使用Spring呢？

从思想层面上，会发现工厂方法模式和IoC/DI的思想是相似的，都是“主动变被动”。但是IOC是通过反射机制来实现的。当我们的需求出现变动时，工厂模式会需要进行相应的变化。但是IOC的反射机制允许我们不重新编译代码，因为它的对象都是动态生成的。

## 4. 什么是AOP？

面向切面编程，可以说是OOP的补充和完善，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。

## 5. BeanFactory 和 FactoryBean的区别？

BeanFactory是IoC最基本的容器，负责生产和管理Bean，它为其他具体的IoC容器提供了最基本的规范。

FactoryBean可以说为IOC容器中Bean的实现提供了更加灵活的方式，当在IOC容器中的Bean实现了FactoryBean后，通过getBean(String BeanName)获取到的Bean对象并不是FactoryBean的实现类对象，而是这个实现类中的getObject()方法返回的对象。要想获取FactoryBean的实现类，就要getBean(&BeanName)，在BeanName之前加上&。

## 6. Spring IOC 的理解，其初始化过程？

1. Resource定位

2. 将Resource定位好的资源载入到BeanDefinition

3. 将BeanDefiniton注册到容器中

BeanDefinition相当于一个数据结构，这个数据结构的生成过程是根据定位的resource资源对象中的bean而来的，这些bean在Spirng IoC容器内部表示成了的BeanDefintion这样的数据结构，IoC容器对bean的管理和依赖注入的实现都是通过操作BeanDefinition来进行的。

## 7. BeanFactory 和 ApplicationContext

在Srping Ioc容器中，有BeanFactory和ApplicationContext两个系列，分别是：

实现BeanFactory接口的简单容器，具备最基本功能。

实现ApplicationContext接口的复杂容器，具备高级功能。

**BeanFactory设计路径**

BeanFactory -> HierarchicalBeanFactory -> ConfigurableBeanFactory，是一个主要的BeanFactory设计路径。

*BeanFactory*：基本规范，比如说getBean()这样的方法。

*HierarchicalBeanFactory*：管理双亲IoC容器规范，比如说getParentBeanFactory()这样的方法。

*ConfigurableBeanFactory*：对BeanFactory的配置功能，比如通过setParentBeanFactory()设置双亲IoC容器，通过addBeanPostProcessor()配置Bean后置处理器。

**ApplicationContext设计路径**

BeanFactory -> ListableBeanFactory 和 HierarchicalBeanFactory -> ApplicationContext -> ConfigurableApplicationContext，是另外一个主要的ApplicationContext设计路径。

*ListableBeanFactory*：细化了许多BeanFactory的功能，比如说getBeanDefinitionNames()。

*ApplicationContext*：通过继承MessageSource、ResourceLoader、ApplicationEventPublisher接口，添加了许多高级特性。

## 8. Spring Bean 的生命周期，如何被管理的？

一个Bean的生命周期活动

* Bean的建立， 由BeanFactory读取Bean定义文件，并生成各个实例
Setter注入，执行Bean的属性依赖注入

* BeanNameAware的setBeanName(), 如果实现该接口，则执行其setBeanName方法

* BeanFactoryAware的setBeanFactory()，如果实现该接口，则执行其setBeanFactory方法

* BeanPostProcessor的processBeforeInitialization()，如果有关联的processor，则在Bean初始化之前都会执行这个实例的processBeforeInitialization()方法

* InitializingBean的afterPropertiesSet()，如果实现了该接口，则执行其afterPropertiesSet()方法

* Bean定义文件中定义init-method

* BeanPostProcessors的processAfterInitialization()，如果有关联的processor，则在Bean初始化之前都会执行这个实例的processAfterInitialization()方法

* DisposableBean的destroy()，在容器关闭时，如果Bean类实现了该接口，则执行它的destroy()方法

* Bean定义文件中定义destroy-method，在容器关闭时，可以在Bean定义文件中使用“destory-method”定义的方法

如果使用ApplicationContext来维护一个Bean的生命周期，则基本上与上边的流程相同，只不过在执行BeanNameAware的setBeanName()后，若有Bean类实现了org.springframework.context.ApplicationContextAware接口，则执行其setApplicationContext()方法，然后再进行BeanPostProcessors的processBeforeInitialization()
实际上，ApplicationContext除了向BeanFactory那样维护容器外，还提供了更加丰富的框架功能，如Bean的消息，事件处理机制等

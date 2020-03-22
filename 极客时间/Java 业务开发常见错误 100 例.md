## 一些知识点

- @Transational生效原则
  - 除非特殊配置（比如使用 AspectJ 静态织入实现 AOP），否则只有定义在 public 方法上的 @Transactional 才能生效。
    - 原因是，Spring     默认通过动态代理的方式实现 AOP，对目标方法进行增强，private 方法无法代理到，Spring 自然也无法动态增强事务处理逻辑。
  - 必须通过代理过的类从外部调用目标方法才能生效。
    - Spring 通过 AOP     技术对方法进行增强，要调用增强过的方法必然是调用代理后的对象。
    - this 指针代表对象自己，Spring 不可能注入 this，所以通过 this 访问方法必然不是代理。
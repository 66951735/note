## 一些知识点

- @Transational生效原则

  - 除非特殊配置（比如使用 AspectJ 静态织入实现 AOP），否则只有定义在 public 方法上的 @Transactional 才能生效。

    - 原因是，Spring     默认通过动态代理的方式实现 AOP，对目标方法进行增强，private 方法无法代理到，Spring 自然也无法动态增强事务处理逻辑。
    - protected方法也不可以，虽然cglib支持protected，但是jdk proxy不支持。

  - 必须通过代理过的类从外部调用目标方法才能生效。（嵌套调用不生效的问题）

    - Spring 通过 AOP技术对方法进行增强，要调用增强过的方法必然是调用代理后的对象。
    - 嵌套调用中外层方法是用this调用的内层方法，而不是代理对象增强后的方法，所以不生效。

  - 默认情况下，出现 RuntimeException（非受检异常）或 Error 的时候，Spring 才会回滚事务。

    - 如果你希望自己捕获异常进行处理的话，也没关系，可以手动设置让当前事务处于回滚状态。

      ```
      @Transactional
      public void test(String name) {
          try {
              // do something
          } catch (Exception ex) {
              log.error("failed", ex);
              TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
          }
      }
      ```

    - 在注解中声明，期望遇到所有的 Exception 都回滚事务（来突破默认不回滚受检异常的限制）

      ```
      @Transactional(rollbackFor = Exception.class)
      ```

      


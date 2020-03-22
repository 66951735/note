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

  - 请注意事务重播机制，根据业务场景采用合适的事务重播机制

    - 默认**PROPAGATION_REQUIRED**，如果外层有事务，则当前事务加入到外层事务中，一起提交，一块回滚。如果外层没有事务，则新建一个事务。默认配置能满足大部分的应用场景
    - 一个默认配置导致事务回滚不服务预期的场景
      - 一个用户注册的操作，会插入一个主用户到用户表，还会注册一个关联的子用户。我们希望将子用户注册的数据库操作作为一个独立事务来处理，即使失败也不会影响主流程，即不影响主用户的注册。
      - 一个常规的写法是，首先子用户注册方法增加transational注解，然后主用户注册完毕，调用子用户方法时，子用户方法放在try-catch代码块中。这样当子用户注册方法抛出异常后由于异常被捕获，所以不会导致主用户注册逻辑回滚。
      - 然而实际上，主用户注册操作却回滚了。这是因为子用户方法执行抛出异常后，当前事务就已经被标记为需要回滚了，默认事务传播机制，导致主用户注册外层事务和子用户注册事务是同一事务，所以主用户注册也会执行回滚操作。
      - 解决方案就是将事务的传播机制变为**REQUIRES_NEW**，子用户注册开启一个新事务，相互隔离不影响。
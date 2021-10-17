# 作用
声明式事务管理

# 使用
@Transactional 注解可以作用于接口、接口方法、类以及类方法上
1. 当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性
2. 当作用在方法级别时会覆盖类级别的定义
3. @Transactional 基于接口的JDK动态代理时，而不是Cglib代理 
4.  当在 protected、private 或者默认可见性的方法上使用 @Transactional 注解时是不会生效的，也不会抛出任何异常 
5.  默认只有来自外部的方法调用才会被AOP代理捕获，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰
6.  Spring 默认只会回滚运行时、未检查异常（继承自 RuntimeException 的异常）或者 Error


# 属性
## readOnly

设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false

## rollbackFor

设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如： 
1. 指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)
2. 指定多个异常类：@Transactional(rollbackFor={RuntimeException.class, BusnessException.class})

## rollbackForClassName

设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。例如： 
1. 指定单一异常类名称：@Transactional(rollbackForClassName=“RuntimeException”) 
2. 指定多个异常类名称：@Transactional(rollbackForClassName={“RuntimeException”,“BusnessException”})

## noRollbackFor

设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚

## noRollbackForClassName
设置不需要进行回滚的异常类名数组

## timeout

设置事务的超时秒数，默认值为-1表示永不超时

## propagation

设置事务的传播行为，默认值为 Propagation.REQUIRED。

事物传播行为介绍:
1.  @Transactional(propagation=Propagation.REQUIRED)：*如果有事务, 那么加入事务, 没有的话新建一个(默认)*
2.  @Transactional(propagation=Propagation.NOT_SUPPORTED)：容器不为这个方法开启事务
3.  @Transactional(propagation=Propagation.REQUIRES_NEW) ：不管是否存在事务,都创建一个新的事务,原来的*挂起*,新的执行完毕,继续执行老的事务；
4.  @Transactional(propagation=Propagation.MANDATORY)： 如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。
5.  @Transactional(propagation=Propagation.NEVER) ：以非事务的方式运行，如果当前存在事务，则抛出异常。
6.  @Transactional(propagation=Propagation.SUPPORTS) ：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
7.  @Transactional(propagation=Propagation.NESTED) ：同 Propagation.REQUIRED 。

## isolation
设置底层数据库的事务隔离级别

事务隔离级别介绍:

1.  @Transactional(isolation = Isolation.READ_UNCOMMITTED)：读取未提交数据(会出现脏读, 不可重复读) 基本不使用
2.  @Transactional(isolation = Isolation.READ_COMMITTED)：读取已提交数据(会出现不可重复读和幻读)
3.  @Transactional(isolation = Isolation.REPEATABLE_READ)：可重复读(MySQL中不会出现幻读，其他数据库有可能)
4.  @Transactional(isolation = Isolation.SERIALIZABLE)：串行化
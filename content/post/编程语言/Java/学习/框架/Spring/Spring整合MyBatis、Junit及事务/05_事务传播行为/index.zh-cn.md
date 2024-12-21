---
title: 05_事务传播行为
description: 05_事务传播行为
date: 2023-01-20
slug: 05_事务传播行为
image: 202412212133331.png
categories:
    - Spring
---

## 5_事务传播行为
### 事务传播行为 propagation
> 当事务方法嵌套调用时，需要控制是否开启新事务，可以使用事务传播行为来控制。
案例：
```java
@Service
public class CountServiceImpl implements CountService {
    @Autowired
    private CountMapper countMapper;
    @Transactional
    @Override
    public void transfer(Integer outUserId, Integer inUserId, Integer money) {
        // 增加
        countMapper.updateMoney(inUserId, money);
        //System.out.println(1/0);
        // 减少
        countMapper.updateMoney(outUserId, -money);
    }
    @Transactional
    @Override
    public void log() {
        System.out.println("记录日志");
        int i = 1/0;
    }
}
```
> 测试方法
```java
@Transactional
public void test(){
    // 转账
    countService.transfer(1,2,10);
    // 记录日志
    countService.log();
}
```
> 假如：转账正常，而记录日志出现异常；则会导致两者都被回滚
| 属性值                         | 行为                                                   |
| ------------------------------ | ------------------------------------------------------ |
| REQUIRED（必须要有）           | 外层方法有事务，内层方法就加入。外层没有，内层就新建   |
| REQUIRES_NEW（必须要有新事务） | 外层方法有事务，内层方法新建。外层没有，内层也新建     |
| SUPPORTS（支持有）             | 外层方法有事务，内层方法就加入。外层没有，内层就也没有 |
| NOT_SUPPORTED（支持没有）      | 外层方法有事务，内层方法没有。外层没有，内层也没有     |
| MANDATORY（强制要求外层有）    | 外层方法有事务，内层方法加入。外层没有。内层就报错     |
| NEVER(绝不允许有)              | 外层方法有事务，内层方法就报错。外层没有。内层就也没有 |
### 隔离级别 isolation
```java
@Transactional(propagation = Propagation.REQUIRES_NEW,isolation = Isolation.READ_COMMITTED)
@Override
public void transfer(Integer outUserId, Integer inUserId, Integer money) {
    // 增加
    countMapper.updateMoney(inUserId, money);
    //System.out.println(1/0);
    // 减少
    countMapper.updateMoney(outUserId, -money);
}
```
| 隔离级别                   | 行为                   |
| -------------------------- | ---------------------- |
| Isolation.DEFAULT          | 使用数据库默认隔离级别 |
| Isolation.READ_COMMITTED   |                        |
| Isolation.READ_UNCOMMITTED |                        |
| Isolation.REPEATABLE_READ  |                        |
| Isolation.SERIALIZABLE     |                        |
### 只读 readOnly
> 如果事务中的操作都是读操作，没涉及到对数据的写操作可以设置readOnly为true。这样可以提高效率。
```java
@Transactional(readOnly = true)
public void log(){
    System.out.println("打印日志");
    int i = 1/0;
}
```
> `注意：`实际项目当中会将日志记录到数据库当中

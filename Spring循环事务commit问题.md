# Spring循环事务commit问题

## 1.抛出问题

在工作中，遇到了一个需求，写一个批量任务，将商品表的一条记录给他赋上多个不同的活动号转移到另外一张活动表里，用以支持不同的活动，其他的不变，我的想法是写一个循环获取活动号数量--->批量任务先删除--->插入--->更新，但是在写好mapper后，在manager层里写事务时，调试发现每次事务只提交了一次，最后是怎么样的一个情况呢，就是insert完了，在执行更新，导致了更新完后，只有一个活动号，这可与我们的需求不符合啊，怎么办呢，我打开了debug,一步步调试，发现明明执行了插入的sql语句，但是打开数据库表更新就是没有数据产生，直到for循环执行完了才能更新到数据库，但这是不可能完成需求的，我原本的想法是执行一次循环--->删除重复数据---->插入数据---->更新数据----->for---->删除数据---****,但是现在直接变成了for--->删除--->插入--->更新，就没了。

## 2.上网查找问题关键

发现是只定义一个对象，所以for循环到最后只会commit一次事务，具体是怎么样的呢，下面是一个例子：

大概是下面这种情况：

```java
 //更新
    @DefaultTransaction
    public void update(updateInAO in){
        ***Mapper.update(in);
    }
    
    
    //删除
    @DefaultTransaction
    public void delete(deleteInAO in){
        ***Mapper.delete(in);
    }
    
    //插入
    @DefaultTransaction
    public void insert(insertInAO in){
        ***Mapper.insert(in);
    }
    //更新数据库操作
    public update***(){
        String[] **id = get**List();
        deleteInAO deleteInAO = new deleteInAO();
        insertInAO insertInAO = new insertInAO();
        updateInAO updateInAO = new updateInAO();
        for(int i=0;i<**id.length;i++){
            //先删除
            deleteInAO.set**();
            deleteInAO.set**();
            delete(deleteInAO);
            //插入
            insertInAO.set**();
            insert(insertInAO);
            //更新
            updateInAO.set**();
            update(updateInAO);
        }
    }
    
    //将一个字符串以逗号分割，获取一个字符串列表
    public String[] get***List(){
        String[] **id = String***.split(",");
        return; **id;
    }
```

刚开始，我们大致是这么写的，由于

```java
  deleteInAO deleteInAO = new deleteInAO();
        insertInAO insertInAO = new insertInAO();
        updateInAO updateInAO = new updateInAO();
```

这里每个对象都只初始化了一次，所以最后事务提交也只会把最后一次的数据commit到事务。

所以我们为了执行一次commit一次，达到我们所需要的需求，接下来进行代码改动。

```java
   //更新
    @DefaultTransaction
    public void update(updateInAO in){
        ***Mapper.update(in);
    }


    //删除
    @DefaultTransaction
    public void delete(deleteInAO in){
        ***Mapper.delete(in);
    }

    //插入
    @DefaultTransaction
    public void insert(insertInAO in){
        ***Mapper.insert(in);
    }
    //更新数据库操作
    public update***(){
        String[] **id = get**List();
        deleteInAO deleteInAO = null;
        insertInAO insertInAO = null;
        updateInAO updateInAO = null;
        try {
            for(int i=0;i<**id.length;i++){
                //先删除
                deleteInAO = new deleteInAO();
                deleteInAO.set**();
                deleteInAO.set**();
                delete(deleteInAO);
                //插入
                insertInAO = new insertInAO();
                insertInAO.set**();
                insert(insertInAO);
                //更新
                updateInAO = new updateInAO();
                updateInAO.set**();
                update(updateInAO);
            }
        }catch(Exception e){
            log.error("e={}",e);
        }

    }

    //将一个字符串以逗号分割，获取一个字符串列表
    public String[] get***List(){
        String[] **id = String***.split(",");
        return; **id;
    }
```

在这段代码里，我们执行了**id.length次初始化，所以事务会提交  * *id.length次，具体是在：

```java
  deleteInAO deleteInAO = null;
        insertInAO insertInAO = null;
        updateInAO updateInAO = null;
        try {
            for(int i=0;i<**id.length;i++){
                //先删除
                deleteInAO = new deleteInAO();
                deleteInAO.set**();
                deleteInAO.set**();
                delete(deleteInAO);
                //插入
                insertInAO = new insertInAO();
                insertInAO.set**();
                insert(insertInAO);
                //更新
                updateInAO = new updateInAO();
                updateInAO.set**();
                update(updateInAO);
            }
```

在这里初始化了多次对象，所以可以提交多次事务。

## 3.更多与此有关的问题

sql执行了但是数据库数据没发生改变：

```

没commit表示数据已经修改了，产生了redo和相应的undo，但是还没有真正生效，别的用户还是看不到你所做的更改。
```

## 4.事务的概念

       事务就是某一组操作，要么都执行，要么都不执行。

       比如你要去银行给朋友转钱，必然存在着从你的账户里扣除一定的金额和向你朋友账户里增加相等的金额这两个操作，这两个操作是不可分割的，无论是哪一个操作的失败，成功的操作也要恢复至最初的状态才能使银行和用户双方都满意，而这样的行为就牵扯到了事务的回滚。

## 5.事务的特性：

       原子性：一个事务是不可分割的最小单位。
       一致性：一个事务在执行之前和执行之后都必须处于一致性状态（比如说转账，前后两个账户的总金额是不会改变的）。
       隔离性：多个并发事务之间的操作不会互相干扰。
       持久性：提交的事务会使得修改的数据是永久的。

## 6.事务的回滚

       回滚：假设A给B转账，A的money--，B的money++,两者是一个整体。只有两个操作都成功完成了，事务才会提交，否则都会回滚到原来的状态。

       在emp表里再添加一个salary属性，把员工编号为1的员工的工资更改为1000，并且判断员工编号为3的员工的工资是否大于5000，如果小于5000则加1000，这两个操作为一个事务，要想事务能够正常的提交的条件是员工编号为3的员工的工资小于5000，工资加了1000，此时员工编号为1的员工的工资更改为1000.如果员工编号为3的员工的工资大于5000则中断程序，抛出异常，即操作2没有顺利完成，事务不应提交，而应该回滚到最初的状态。

       当然，不是所有的时候都需要恢复最初的状态的，所以有时候就会用到自己设置回滚点，上述程序中注释的部分就是自己添加了如果事务没有正常提交想要恢复到程序执行到哪个位置的回滚点。

## 7.总结

如果spring循环事务处理只需要一次cmomit的，只需要初始化一次对象就行了，如果需要多次commit事务的，就需要多次初始化对象。

## 8.致谢

感谢您的阅读，对于错误的地方，请在评论区指出来，博主好进行更正，谢谢！
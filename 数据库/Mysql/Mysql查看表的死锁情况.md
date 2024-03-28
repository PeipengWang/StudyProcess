# msyql查看表死锁情况
1、在做数据库操作时，有时会因为自己的粗心或者程序设计上的缺陷导致锁表，在mysql中查看锁表和解锁的步骤如下：  
    //1.查看当前数据库锁表的情况   
    SELECT * FROM information_schema.INNODB_TRX;   
    //2.杀掉查询结果中锁表的trx_mysql_thread_id    
    kill trx_mysql_thread_id  

2、另外一种查询锁方法  
    1、查询是否锁表   
    show OPEN TABLES where In_use > 0;    
    2、查询进程   
    show processlist   
    查询到相对应的进程===然后 kill    id   
    补充：   
    查看正在锁的事务   
    SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;    
    查看等待锁的事务   
    SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;   

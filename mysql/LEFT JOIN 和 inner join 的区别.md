# LEFT JOIN 和 inner join 的区别



student表

![image-20200916002855999](https://gitee.com/fking86/images4typora/raw/master/imgs/20200916002905.png)



sc 表

![image-20200916003333770](https://gitee.com/fking86/images4typora/raw/master/imgs/20200916003333.png)

首先where条件a.Sid = b.Sid 查询

SELECT * FROM student a,sc b WHERE a.Sid = b.Sid GROUP BY a.Sname ORDER BY a.Sid

结果：（from后用‘，’分隔，两表inner join 搜索出a,b表都有的数据）

![image-20200916003358131](https://gitee.com/fking86/images4typora/raw/master/imgs/20200916003358.png)



left join 条件查询

select * from student a LEFT JOIN sc b ON a.Sid = b.Sid GROUP BY a.Sname order BY a.Sid

结果：（left join 连接，左表数据全部+右表符合on条件的数据。left join 左右表互换结果不一样）

![image-20200916003419873](https://gitee.com/fking86/images4typora/raw/master/imgs/20200916003420.png)

附一张图：

![image-20200916003506321](https://gitee.com/fking86/images4typora/raw/master/imgs/20200916003506.png)
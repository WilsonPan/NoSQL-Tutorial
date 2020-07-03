# 慢查询

## 开启慢查询 && 获取慢查询语句

- 获取当前监控状态
```
db.getProfilingStatus()
```

- 设置监控
```
db.setProfilingLevel(level，options)
```
**level**  |  integer   |   profiler level
 
> 0 : 关闭日志收集  
> 1 : 收集大于slowms的日志  
> 2 : 收集所有日志

**options**  |  document/integer  |  Optional

> 若是整数, 则赋值给slowms  
> 若是对象 , 则是  
> slowms : 阈值 Default: 100 , 单位毫秒(milliseconds)  
> sampleRate : 采样率 Default: 1.0 , eg. 1 记录所有慢查询 , 0.5 记录50%的慢查询  


- 获取慢查询

```cs
// 按执行时间排序获取前20条
db.system.profile.find().sort( { millis : -1 } ).limit(20).pretty();

// 获取最近20条执行数据
db.system.profile.find().limit(20).sort( { ts : -1 } ).pretty();

// 获取大于100ms的查询的20条数据
db.system.profile.find( { millis : { $gt : 100 } } ).limit(20).pretty()

// 获取时间范围段
db.system.profile.find({
  ts : {
    $gt: new ISODate("2020-06-09T03:00:00Z"),
    $lt: new ISODate("2020-06-09T03:40:00Z")
  }
}).limit(20).pretty()
```

**注意**

> 分析可能会影响性能并与系统日志共享设置。在生产部署上配置和启用探查器之前，请仔细考虑所有性能和安全隐患。

## 扩展

日志保存在system.profile集合中, 这是一个固定集合, 默认大小:1M.  

若要修改system.profile固定集合大小 , 可按下面4步
1. 禁止日志记录
2. 删除system.profile集合
3. 创建一个新system.profile 集合
4. 重新启用分析

例如 : 创建一个4 MB大小的日志收集集合
shell
```
db.setProfilingLevel(0)

db.system.profile.drop()

db.createCollection( "system.profile", { capped: true, size:4000000 } )

db.setProfilingLevel(1)
```

## 参考文章
[https://docs.mongodb.com/manual/reference/command/profile/](https://docs.mongodb.com/manual/reference/command/profile/)  
[https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/#database-profiling-overhead](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/#database-profiling-overhead)
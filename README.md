# crontab

golang 实现分布式任务调度系统

## 依赖

etcd、mongodb

### etcd

https://github.com/etcd-io/etcd

### mongodb

https://github.com/mongodb/mongo

## 架构

### master

![image](https://github.com/user-attachments/assets/9d247be7-0719-4ee4-8580-fde36405cd3f)

### worker

![image](https://github.com/user-attachments/assets/a0222337-c114-419c-87f8-380f45000c9e)

#### 监听协程

利用 watch api，监听 /cron/jobs/ 和 /cron/killer/ 目录的变化

将变化时间通过 channel 推送给调度协程，更新内存中的任务信息

#### 调度协程

监听任务变更 event，更新内存中维护的任务列表

检查任务 cron 表达式，扫描到期任务，交给执行协程运行

监听任务控制 event，强制中断正在执行中的子进程

监听任务执行 result，更新内存中任务状态，投递执行日志

#### 执行协程

在 etcd 中抢占分布式乐观锁：/cron/lock/任务名

抢占成功则通过 Command 类执行 shell 任务

捕获 Command 输出并等待子进程结束，将执行结果投递给调度协程

备注：公平抢占依赖服务器时间同步

#### 日志协程

监听调度协程发来的执行日志，放入一个 batch 中

对新 batch 启动定时器，超时未满自动提交

若 batch 被放满，那么立即提交，并取消自动提交定时器

## 效果

![crontab](./images/crontab.png)

## 致谢

https://yuerblog.cc

## TODO

- [ ] 任务超时控制
- [ ] 任务执行错误/超时进行告警

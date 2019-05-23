---
title: APSchedule 简介
tags:APSchedule
---
## Python任务调度模块 – APScheduler
#### APScheduler是一个Python定时任务框架，使用起来十分方便。提供了基于日期、固定时间间隔以及crontab类型的任务，并且可以持久化任务、并以daemon方式运行应用。目前最新版本为3.0.x。
## 四个组件：
### 触发器(trigger)
包含调度逻辑，每一个作业有它自己的触发器，用于决定接下来哪一个作业会运行。除了他们自己初始配置意外，触发器完全是无状态的。
### 作业存储(job store)
存储被调度的作业，默认的作业存储是简单地把作业保存在内存中，其他的作业存储是将作业保存在数据库中。一个作业的数据讲在保存在持久化作业存储时被序列化，并在加载时被反序列化。调度器不能分享同一个作业存储。
### 执行器(executor)
处理作业的运行，他们通常通过在作业中提交制定的可调用对象到一个线程或者进城池来进行。当作业完成时，执行器将会通知调度器。
### 调度器(scheduler)
是其他的组成部分。你通常在应用只有一个调度器，应用的开发者通常不会直接处理作业存储、调度器和触发器，相反，调度器提供了处理这些的合适的接口。配置作业存储和执行器可以在调度器中完成，例如添加、修改和移除作业。
你需要选择合适的调度器，这取决于你的应用环境和你使用APScheduler的目的。通常最常用的两个：  

> BlockingScheduler: 当调度器是你应用中唯一要运行的东西时使用。
 >BackgroundScheduler: 当你不运行任何其他框架时使用，并希望调度器在你应用的后台执行。
 
 
安装APScheduler非常简单：
> pip install apscheduler

选择合适的作业存储，你需要决定是否需要作业持久化。如果你总是在应用开始时重建job，你可以直接使用默认的作业存储（MemoryJobStore).但是如果你需要将你的作业持久化，以避免应用崩溃和调度器重启时，你可以根据你的应用环境来选择具体的作业存储。例如：使用Mongo或者SQLAlchemyJobStore （用于支持大多数RDBMS）
然而，调度器的选择通常是为你如果你使用上面的框架之一。然而，默认的ThreadPoolExecutor 通常用于大多数用途。如果你的工作负载中有较大的CPU密集型操作，你可以考虑用ProcessPoolExecutor来使用更多的CPU核。你也可以在同一时间使用两者，将进程池调度器作为第二执行器。

### 配置调度器
APScheduler提供了许多不同的方式来配置调度器，你可以使用一个配置字典或者作为参数关键字的方式传入。你也可以先创建调度器，再配置和添加作业，这样你可以在不同的环境中得到更大的灵活性。
下面是一个简单使用BlockingScheduler，并使用默认内存存储和默认执行器。(默认选项分别是MemoryJobStore和ThreadPoolExecutor，其中线程池的最大线程数为10)。配置完成后使用start()方法来启动。

``` python
from apscheduler.schedulers.blocking import BlockingScheduler
def my_job():
    print 'hello world'

sched = BlockingScheduler()
sched.add_job(my_job, 'interval', seconds=5)
sched.start()

from apscheduler.schedulers.blocking import BlockingScheduler
def my_job():
    print 'hello world'
 
sched = BlockingScheduler()
sched.add_job(my_job, 'interval', seconds=5)
sched.start()
```
在运行程序5秒后，将会输出第一个Hello world。
下面进行一个更复杂的配置，使用两个作业存储和两个调度器。在这个配置中，作业将使用mongo作业存储，信息写入到MongoDB中。

```python

from pymongo import MongoClient
from apscheduler.schedulers.blocking import BlockingScheduler
from apscheduler.jobstores.mongodb import MongoDBJobStore
from apscheduler.jobstores.memory import MemoryJobStore
from apscheduler.executors.pool import ThreadPoolExecutor, ProcessPoolExecutor


def my_job():
    print 'hello world'
host = '127.0.0.1'
port = 27017
client = MongoClient(host, port)

jobstores = {
    'mongo': MongoDBJobStore(collection='job', database='test', client=client),
    'default': MemoryJobStore()
}
executors = {
    'default': ThreadPoolExecutor(10),
    'processpool': ProcessPoolExecutor(3)
}
job_defaults = {
    'coalesce': False,
    'max_instances': 3
}
scheduler = BlockingScheduler(jobstores=jobstores, executors=executors, job_defaults=job_defaults)
scheduler.add_job(my_job, 'interval', seconds=5)

try:
    scheduler.start()
except SystemExit:
    client.close()
```

```python
from apscheduler.schedulers.blocking import BlockingScheduler
from apscheduler.jobstores.mongodb import MongoDBJobStore
from apscheduler.jobstores.memory import MemoryJobStore
from apscheduler.executors.pool import ThreadPoolExecutor, ProcessPoolExecutor
 
 
def my_job():
    print 'hello world'
host = '127.0.0.1'
port = 27017
client = MongoClient(host, port)
 
jobstores = {
    'mongo': MongoDBJobStore(collection='job', database='test', client=client),
    'default': MemoryJobStore()
}
executors = {
    'default': ThreadPoolExecutor(10),
    'processpool': ProcessPoolExecutor(3)
}
job_defaults = {
    'coalesce': False,
    'max_instances': 3
}
scheduler = BlockingScheduler(jobstores=jobstores, executors=executors, job_defaults=job_defaults)
scheduler.add_job(my_job, 'interval', seconds=5)
 
try:
    scheduler.start()
except SystemExit:
    client.close()
```
查询MongoDB可以看到作业的运行情况如下：

``` json
{ 
  "_id" : "55ca54ee4bb744f8a5ab08cc4319bc24",
  "next_run_time" : 1434017278.797,
  "job_state" : new BinData(0, "gAJ9cQEoVQRhcmdzcQIpVQhleGVjdXRvcnEDVQdkZWZhdWx0cQRVDW1heF9pbnN0YW5jZXNxBUsDVQRmdW5jcQZVD19fbWFpbl9fOm15X2pvYnEHVQJpZHEIVSA1NWNhNTRlZTRiYjc0NGY4YTVhYjA4Y2M0MzE5YmMyNHEJVQ1uZXh0X3J1bl90aW1lcQpjZGF0ZXRpbWUKZGF0ZXRpbWUKcQtVCgffBgsSBzoMKUhjcHl0egpfcApxDChVDUFzaWEvU2hhbmdoYWlxDU2AcEsAVQNDU1RxDnRScQ+GUnEQVQRuYW1lcRFVBm15X2pvYnESVRJtaXNmaXJlX2dyYWNlX3RpbWVxE0sBVQd0cmlnZ2VycRRjYXBzY2hlZHVsZXIudHJpZ2dlcnMuaW50ZXJ2YWwKSW50ZXJ2YWxUcmlnZ2VyCnEVKYFxFn1xF1UPaW50ZXJ2YWxfbGVuZ3RocRhHQBQAAAAAAABzfXEZKFUIdGltZXpvbmVxGmgMKGgNTehxSwBVA0xNVHEbdFJxHFUIaW50ZXJ2YWxxHWNkYXRldGltZQp0aW1lZGVsdGEKcR5LAEsFSwCHUnEfVQpzdGFydF9kYXRlcSBoC1UKB98GCxIHIQwpSGgPhlJxIVUIZW5kX2RhdGVxIk51hmJVCGNvYWxlc2NlcSOJVQd2ZXJzaW9ucSRLAVUGa3dhcmdzcSV9cSZ1Lg==")
}
{ 
  "_id" : "55ca54ee4bb744f8a5ab08cc4319bc24",
  "next_run_time" : 1434017278.797,
  "job_state" : new BinData(0, "gAJ9cQEoVQRhcmdzcQIpVQhleGVjdXRvcnEDVQdkZWZhdWx0cQRVDW1heF9pbnN0YW5jZXNxBUsDVQRmdW5jcQZVD19fbWFpbl9fOm15X2pvYnEHVQJpZHEIVSA1NWNhNTRlZTRiYjc0NGY4YTVhYjA4Y2M0MzE5YmMyNHEJVQ1uZXh0X3J1bl90aW1lcQpjZGF0ZXRpbWUKZGF0ZXRpbWUKcQtVCgffBgsSBzoMKUhjcHl0egpfcApxDChVDUFzaWEvU2hhbmdoYWlxDU2AcEsAVQNDU1RxDnRScQ+GUnEQVQRuYW1lcRFVBm15X2pvYnESVRJtaXNmaXJlX2dyYWNlX3RpbWVxE0sBVQd0cmlnZ2VycRRjYXBzY2hlZHVsZXIudHJpZ2dlcnMuaW50ZXJ2YWwKSW50ZXJ2YWxUcmlnZ2VyCnEVKYFxFn1xF1UPaW50ZXJ2YWxfbGVuZ3RocRhHQBQAAAAAAABzfXEZKFUIdGltZXpvbmVxGmgMKGgNTehxSwBVA0xNVHEbdFJxHFUIaW50ZXJ2YWxxHWNkYXRldGltZQp0aW1lZGVsdGEKcR5LAEsFSwCHUnEfVQpzdGFydF9kYXRlcSBoC1UKB98GCxIHIQwpSGgPhlJxIVUIZW5kX2RhdGVxIk51hmJVCGNvYWxlc2NlcSOJVQd2ZXJzaW9ucSRLAVUGa3dhcmdzcSV9cSZ1Lg==")
}

```
# 操作作业

```python
 # 添加作业:
 @sched.scheduled_job('cron', id='my_job_id', day='last sun')
 def some_decorated_task():
	 pass
job.remove()
scheduler.remove_job('my_job_id')
# 暂停作业:
apscheduler.job.Job.pause()
apscheduler.schedulers.base.BaseScheduler.pause_job()
# 恢复作业:
apscheduler.job.Job.resume()
apscheduler.schedulers.base.BaseScheduler.resume_job()
# 获得job列表
get_jobs()
print_jobs()
# 修改作业
def some_decorated_task():
    print("I am printed at 00:00:00 on the last Sunday of every month!") 
# 关闭作业
scheduler.shutdown()
scheduler.shutdown(wait=False)
# 作业运行的控制
# add_job的第二个参数是trigger，
# 它管理着作业的调度方式。
# 它可以为date, interval或者cron。对于不同的trigger，对应的参数也相同。

(1). cron定时调度
year (int|str) – 4-digit year
month (int|str) – month (1-12)
day (int|str) – day of the (1-31)
week (int|str) – ISO week (1-53)
day_of_week (int|str) – number or name of weekday (0-6 or mon,tue,wed,thu,fri,sat,sun)
hour (int|str) – hour (0-23)
minute (int|str) – minute (0-59)
second (int|str) – second (0-59)
start_date (datetime|str) – earliest possible date/time to trigger on (inclusive)
end_date (datetime|str) – latest possible date/time to trigger on (inclusive)
timezone (datetime.tzinfo|str) – time zone to use for the date/time calculations (defaults to scheduler timezone)
和Linux的Crontab一样，它的值格式为：

几个例子如下：


sched.add_job(job_function, 'cron', month='6-8,11-12', day='3rd fri', hour='0-3')
sched.add_job(job_function, 'cron', day_of_week='mon-fri', hour=5, minute=30, end_date='2014-05-30')
sched.add_job(job_function, 'cron', month='6-8,11-12', day='3rd fri', hour='0-3')
sched.add_job(job_function, 'cron', day_of_week='mon-fri', hour=5, minute=30, end_date='2014-05-30')
(2). interval 间隔调度
它的参数如下：
weeks (int) – number of weeks to wait
days (int) – number of days to wait
hours (int) – number of hours to wait
minutes (int) – number of minutes to wait
seconds (int) – number of seconds to wait
start_date (datetime|str) – starting point for the interval calculation
end_date (datetime|str) – latest possible date/time to trigger on
timezone (datetime.tzinfo|str) – time zone to use for the date/time calculations
例子：

sched.add_job(job_function, 'interval', hours=2)
sched.add_job(job_function, 'interval', hours=2)
(3). date 定时调度
最基本的一种调度，作业只会执行一次。它的参数如下：
run_date (datetime|str) – the date/time to run the job at
timezone (datetime.tzinfo|str) – time zone for run_date if it doesn’t have one already
例子：
sched.add_job(my_job, 'date', run_date=date(2009, 11, 6), args=['text'])
sched.add_job(my_job, 'date', run_date=datetime(2009, 11, 6, 16, 30, 5), args=['text'])
^^
```
Ref:
> http://apscheduler.readthedocs.org/en/latest/modules/triggers
> http://apscheduler.readthedocs.org/en/3.0/userguide.html


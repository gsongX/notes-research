---
title: 2017-2-10 Plan 设置 crontab python 接口
tags: 
grammar_cjkRuby: true
---
```python

from plan import Plan

cron = Plan()

cron.command('ls /tmp', every='1.day', at='12:00')
cron.command('pwd', every='2.month')
cron.command('date', every='weekend')

if __name__ == "__main__":
    cron.run()

```
---
title: open-falcon监控系统调研
tags: open-falcon,openTsdb
grammar_cjkRuby: true
---

## 采集格式 类似于openTSDB

### TSdb 相关链接

TSDB http://liubin.org/blog/2016/02/18/tsdb-intro/
openTSDB http://liubin.org/blog/2016/03/05/tsdb-opentsdb/


### openfalcon 采集格式示例

``` groovy
<
Endpoint:yunpiao, 
Metric:snmp.Udp.RcvbufErrors, 
Tags:map[], 
Value:0, 
TS:1476770550 2016-10-18 14:02:30 
DsType:, 
Step:0, 
Heartbeat:0
>
```

### 相关技术

1. 采用心跳包,判断是否在线
2. 为控制洪流,采用内存定长queue,整理数据 
3. 采用一致性哈希判断所属实例

## open-falcon 信息采集项
~~~
Metric:agent.alive
Metric:cpu.idle
Metric:cpu.user
Metric:cpu.nice
Metric:cpu.system
Metric:cpu.iowait
Metric:cpu.irq
Metric:cpu.softirq
Metric:cpu.steal
Metric:cpu.guest
Metric:cpu.switches
Metric:kernel.maxfiles
Metric:kernel.maxproc
Metric:kernel.files.allocated
Metric:kernel.files.left
Metric:load.1min
Metric:load.5min
Metric:load.15min
Metric:mem.memtotal
Metric:mem.memfree.percent
Metric:mem.swapfree.percent
Metric:mem.swapused.percent
Metric:disk.io.read_requests
Metric:disk.io.read_merged
Metric:disk.io.read_sectors
Metric:disk.io.msec_read
Metric:disk.io.write_requests
Metric:disk.io.write_merged
Metric:disk.io.write_sectors
Metric:disk.io.msec_write
Metric:disk.io.ios_in_progress
Metric:disk.io.msec_total
Metric:disk.io.msec_weighted_total
Metric:disk.io.read_requests
Metric:disk.io.read_merged
Metric:disk.io.read_sectors
Metric:disk.io.msec_read
Metric:disk.io.write_requests
Metric:disk.io.write_merged
Metric:disk.io.write_sectors
Metric:disk.io.msec_write
Metric:disk.io.ios_in_progress
Metric:disk.io.msec_total
Metric:disk.io.msec_weighted_total
Metric:disk.io.read_bytes
Metric:disk.io.write_bytes
Metric:disk.io.avgrq_sz
Metric:disk.io.avgqu-sz
Metric:disk.io.await
Metric:disk.io.svctm
Metric:disk.io.util
Metric:disk.io.read_bytes
Metric:disk.io.write_bytes
Metric:disk.io.avgrq_sz
Metric:disk.io.avgqu-sz
Metric:disk.io.await
Metric:disk.io.svctm
Metric:disk.io.util
Metric:TcpExt.TCPFastRetrans
Metric:TcpExt.DelayedACKLocked
Metric:TcpExt.TCPDSACKUndo
Metric:TcpExt.TCPLostRetransmit
Metric:TcpExt.TCPPrequeueDropped
Metric:TcpExt.TCPTSReorder
Metric:TcpExt.TCPAbortOnTimeout
Metric:TcpExt.TW
Metric:TcpExt.ArpFilter
Metric:TcpExt.TCPLossFailures
Metric:TcpExt.TCPSchedulerFailed
Metric:TcpExt.LockDroppedIcmps
Metric:TcpExt.TCPAbortOnMemory
Metric:TcpExt.TCPBacklogDrop
Metric:TcpExt.TCPMinTTLDrop
Metric:TcpExt.TCPMemoryPressures
Metric:TcpExt.TCPAbortFailed
Metric:TcpExt.TCPSpuriousRTOs
Metric:TcpExt.ListenOverflows
Metric:TcpExt.ListenDrops
Metric:TcpExt.PruneCalled
Metric:TcpExt.TCPTimeouts
Metric:snmp.Udp.InErrors
Metric:snmp.Udp.OutDatagrams
Metric:snmp.Udp.RcvbufErrors
Metric:snmp.Udp.SndbufErrors
Metric:snmp.Udp.InCsumErrors
Metric:snmp.Udp.IgnoredMulti
Metric:snmp.Udp.InDatagrams
Metric:snmp.Udp.NoPorts
Metric:ss.closed
Metric:ss.orphaned
Metric:ss.synrecv
Metric:ss.timewait
Metric:ss.slabinfo.timewait
Metric:ss.estab
Metric:df.bytes.free.percent
Metric:df.inodes.free.percent
Metric:df.bytes.free.percent
Metric:df.inodes.free.percent
Metric:df.statistics.total
Metric:df.statistics.used
Metric:df.statistics.used.percent
~~~

















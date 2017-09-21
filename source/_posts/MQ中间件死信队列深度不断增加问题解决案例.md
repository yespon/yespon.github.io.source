---
title: MQ中间件死信队列深度不断增加问题解决案例
date: 2017-09-21 13:29:44
category:
    - 架构&设计
    - 经典文摘
tags:
    - MQ
    - 消息中间件
---
**背景：工行某分行发现小额MQ死信队列深度已超过1W，而且还一直在增加，但报文发送、接收均正常。**

问题排查过程
1、检查应用日志、mq发送日志，均未发现异常。

2、查看mq死信队列信息

```
bash-3.2$ ./amqsbcg DEADQ QMMBFE
AMQSBCG0 - starts here
```


```
**********************
 MQOPEN - 'DEADQ'
 MQGET of message number 1 
****Message descriptor****
  StrucId  : 'MD  '  Version : 2
  Report   : 0  MsgType : 8
  Expiry   : -1  Feedback : 0
  Encoding : 273  CodedCharSetId : 819
  Format : 'MQDEAD  '
  Priority : 0  Persistence : 0
  MsgId : X'414D5120514D4D424645202020202020505376A520000802'
  CorrelId : X'000000000000000000000000000000000000000000000000'
  BackoutCount : 0
  ReplyToQ       : '                                                '
  ReplyToQMgr    : 'QMMBFE                                          '
  ** Identity Context
  UserIdentifier : '            '
  AccountingToken : 
   X'0000000000000000000000000000000000000000000000000000000000000000'
  ApplIdentityData : '                                '
  ** Origin Context
  PutApplType    : '7'
  PutApplName    : 'QMMBFE                      '
  PutDate  : '20120914'    PutTime  : '18260760'
  ApplOriginData : '    '
  GroupId : X'000000000000000000000000000000000000000000000000'
  MsgSeqNumber   : '1'
  Offset         : '0'
  MsgFlags       : '0'
  OriginalLength : '-1'
****   Message      ****
 length - 856 bytes
00000000:  444C 4820 0000 0001 0000 0109 5359 5354 'DLH ........SYST'
00000010:  454D 2E43 4943 532E 494E 4954 4941 5449 'EM.CICS.INITIATI'
00000020:  4F4E 2E51 5545 5545 2020 2020 2020 2020 'ON.QUEUE        '
00000030:  2020 2020 2020 2020 2020 2020 514D 4D42 '            QMMB'
00000040:  4645 2020 2020 2020 2020 2020 2020 2020 'FE              '
00000050:  2020 2020 2020 2020 2020 2020 2020 2020 '                '
00000060:  2020 2020 2020 2020 2020 2020 0000 0111 '            ....'
00000070:  0000 0333 4D51 5452 4947 2020 0000 0006 '...3MQTRIG  ....'
00000080:  5255 4E4D 5154 524D 0000 0000 0000 0000 'RUNMQTRM........'
00000090:  0000 0000 0000 0000 0000 0000 3230 3132 '............2012'
000000A0:  3039 3134 3138 3236 3037 3633 544D 2020 '091418260763TM  '
000000B0:  0000 0001 3130 3238 3831 3030 3030 3139 '....102881000019'
000000C0:  5F32 2020 2020 2020 2020 2020 2020 2020 '_2              '
000000D0:  2020 2020 2020 2020 2020 2020 2020 2020 '                '
000000E0:  2020 2020 554E 4958 2E50 524F 3220 2020 '    UNIX.PRO2   '
000000F0:  2020 2020 2020 2020 2020 2020 2020 2020 '                '
00000100:  2020 2020 2020 2020 2020 2020 2020 2020 '                '
00000110:  2020 2020 2020 2020 2020 2020 2020 2020 '                '
00000120:  2020 2020 2020 2020 2020 2020 2020 2020 '                '
00000130:  2020 2020 2020 2020 2020 2020 2020 2020 '                '
00000140:  2020 2020 2020 2020 2020 2020 2020 2020 '                '
00000150:  2020 2020 0000 0006 2F62 6570 736D 6266 '    ..../bepsmbf'
00000160:  652F 6269 6E2F 6C69 622F 4D51 6372 6563 'e/bin/lib/MQcrec'
00000170:  7620 2020 2020 2020 2020 2020 2020 2020 'v               '
```

3、检查MQDLQ结构如下
 
```
MQDLH
```

4、查到其ReasonCode为 0000 0109

5、X'00000109' 含义为：MQFB_APPL_CANNOT_BE_STARTED
Application cannot be started.
An application processing a trigger message was unable to start the
application named in theApplIdfield of the trigger message.

6、也就是说如下程序无法启动

```
/bepsmbfe/bin/lib/MQcrecv
/bepsmbfe/bin/lib/MQrrecv
```

7、启动不了的可能原因：文件不存在、没有执行权限等。
了解到其真实原因为路径错误（正确路径为/home/bepsmbfe/bin/lib/）,将其路径改正确即可解决问题。

8、解决问题的步骤
第1步: 停止相关应用
第2步: 重新定义process

```
#su - mqm
$runmqsc QMMBFE 
DEF PROCESS(unix.pro1) APPLTYPE(UNIX) APPLICID('/home/bepsmbfe/bin/lib/MQrrecv') REPLACE
DEF PROCESS(unix.pro2) APPLTYPE(UNIX) APPLICID('/home/bepsmbfe/bin/lib/MQcrecv') REPLACE
```

第3步：启动相关应用
第4步：继续观察死信队列的状况，发现不再增加，问题解决。
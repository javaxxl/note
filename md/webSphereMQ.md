MQCHNRDSS01

crtmqm MQCHNRDSS01
strmqm MQCHNRDSS01

runmqsc MQCHNRDSS01 << !
ALTER QMGR +
MAXMSGL(6291456) +
CCSID(1208) +
DEADQ('MQCHNRDSS01.DEAD.QUEUE') +
REPOS('CHNRDSSCLUSTER') +
CHLAUTH(DISABLED) +
FORCE

ALTER CHANNEL(SYSTEM.DEF.SVRCONN) +
CHLTYPE(SVRCONN) +
MCAUSER('mqm')
END
!
*监听器，用户开放系统的通道连接，端口号可以自定义
runmqsc MQCHNRDSS01 << !
DEFINE LISTENER('MQCHNRDSSLSR') +
TRPTYPE(TCP) +
PORT(11000) +
BACKLOG(0) +
CONTROL (QMGR) +
REPLACE
START LISTENER('MQCHNRDSSLSR')
* 死信队列
DEFINE QLOCAL('MQCHNRDSS01.DEAD.QUEUE') +
DESCR(' ') +
DEFPSIST(YES) +
MAXDEPTH(999999999) +
MAXMSGL(104857600) +
REPLACE
* 集群发送通道，其中连接名是通道名中队列管理器的ip和端口
DEFINE CHANNEL('TO.MQCHNRDSS11') +
CHLTYPE(CLUSSDR) +
TRPTYPE(TCP) +
CLUSTER('CHNRDSSCLUSTER') +
CONNAME('192.168.3.49(11001)') +
DISCINT(0) +
MAXMSGL(6291456) +
REPLACE
START CHANNEL(TO.MQCHNRDSS11)
DIS CHS(TO.MQCHNRDSS11)
* 集群接受通道，其中连接名是通道名中队列管理器的ip和端口
DEFINE CHANNEL('TO.MQCHNRDSS01') +
CHLTYPE(CLUSRCVR) +
TRPTYPE(TCP) +
CLUSTER('CHNRDSSCLUSTER') +
CONNAME('192.168.3.49(11000)') +
DISCINT(0) +
MAXMSGL(6291456) +
REPLACE
START CHANNEL(TO.MQCHNRDSS01)
DIS CHS(TO.MQCHNRDSS01)
* 队列管理器别名
DEFINE QREMOTE('MQCHNRDSS_ALIAS') +
DESCR(' ') +
REPLACE

* 传输队列
DEFINE QLOCAL('MQEGSPGW') +
DESCR(' ') +
DEFPSIST(NO) +
MAXDEPTH(999999999) +
MAXMSGL(104857600) +
USAGE(XMITQ) +
REPLACE

* 发送通道需要正确配置连接名中对方队列管理器ip和端口
DEFINE CHANNEL('CHNRDSS.EGSPGW') +
CHLTYPE(SDR) +
TRPTYPE(TCP) +
CONNAME('192.168.3.53(20001)') +
DISCINT(0) +
SHORTTMR(20) +
SHORTRTY(100) +
LONGTMR(600) +
LONGRTY(999999999) +
MAXMSGL(6291456) +
XMITQ('EGSPGW') +
REPLACE

START CHANNEL(CHNRDSS.EGSPGW)
DIS CHS(CHNRDSS.EGSPGW)
* 接受通道，与对方的发送通道构成一组完整的通道
DEFINE CHANNEL('EGSPGW.CHNRDSS') +
CHLTYPE(RCVR) +
TRPTYPE(TCP) +
MAXMSGL(6291456) +
REPLACE
START CHANNEL (EGSPGW.CHNRDSS)
DIS CHS (EGSPGW.CHNRDSS)

* 远程返回队列，用于发送消息
DEFINE QREMOTE('REMOTEQ.EGSP01.RSP') +
CLUSTER('CHNRDSSCLUSTER') +
DEFBIND(NOTFIXED) +
DESCR(' ') +
RQMNAME('MQEGEPGW_ALIAS') +
RNAME('LOCALQ.P.CHNRDSS01.RSP') +
XMITQ('MQEGSPGW') +
REPLACE
* 远程请求队列，用于发送请求报文
DEFINE QREMOTE(REMOTEQ.EGSP01.REQ) +
CLUSTER('CHNRDSSCLUSTER') +
DEFBIND(NOTFIXED) +
DESCR(' ') +
RQMNAME('MQEGSPGW_ALIAS') +
RNAME('LOCALQ.C.CHNRDSS01.REQ') +
XMITQ('MQEGSPGW') +
REPLACE
* 本地返回队列用于接受返回报文，应用或代理从这个队列中读取服务方的返回报文
DEFINE QLOCAL('LOCALQ.CHNRDSS01.RSP') +
DESCR(' ') +
DEFPSIST(NO) +
MAXDEPTH(999999999) +
MAXMSGL(6291456) +
CLUSTER('CHNRDSSCLUSTER') +
DEFBIND(NOTFIXED) +
REPLACE
END
!

MQCHNRDSS11
crtmqm MQCHNRDSS11
strmqm MQCHNRDSS11
runmqsc MQCHNRDSS11 << !
ALTER QMGR +
MAXMSGL(6291456) +
CCSID(1208) +
DEADQ('MQCHNRDSS11.DEAD.QUEUE') +
REPOS('CHNRDSSCLUSTER') +
CHLAUTH(DISABLED) +
FORCE

ALTER CHANNEL(SYSTEM.DEF.SVRCONN) +
CHLTYPE(SVRCONN) +
MCAUSER('mqm')
END
!

runmqsc MQCHNRDSS11 << !
* lister
DEFINE LISTENER('MQCHNRDSS11LSR') +
TRPTYPE(TCP) +
PORT(11001) +
BACKLOG(0) +
CONTROL (QMGR) +
REPLACE
START LISTENER('MQCHNRDSS11LSR')
* 死信队列
DEFINE QLOCAL('MQCHNRDSS11.DEAD.QUEUE') +
DESCR(' ') +
DEFPSIST(YES) +
MAXDEPTH(999999999) +
MAXMSGL(104857600) +
REPLACE
* 集群发送通道，其中连接名是通道名中队列管理器的ip和端口
DEFINE CHANNEL('TO.MQCHNRDSS01') +
CHLTYPE(CLUSSDR) +
TRPTYPE(TCP) +
CLUSTER('CHNRDSSCLUSTER') +
CONNAME('192.168.3.49(11000)') +
DISCINT(0) +
MAXMSGL(6291456) +
REPLACE
START CHANNEL(TO.MQCHNRDSS01)
DIS CHS(TO.MQCHNRDSS01)
* 集群接受通道，其中连接名是通道名中队列管理器的ip和端口
DEFINE CHANNEL('TO.MQCHNRDSS11') +
CHLTYPE(CLUSRCVR) +
TRPTYPE(TCP) +
CLUSTER('CHNRDSSCLUSTER') +
CONNAME('192.168.3.49(11001)') +
DISCINT(0) +
MAXMSGL(6291456) +
REPLACE
START CHANNEL(TO.MQCHNRDSS11)
DIS CHS(TO.MQCHNRDSS11)
* 本地请求队列，用于接受请求报文，应用或代理从这个队列中读取请求方的报文
DEFINE QLOCAL('LOCALQ.CHNRDSS01.REQ') +
DESCR(' ') +
MAXDEPTH(999999999) +
MAXMSGL(6291456) +
CLUSTER('CHNRDSSCLUSTER') +
DEFBIND(NOTFIXED) +
REPLACE
END
!


MQJJH01
crtmqm MQJJH01
strmqm MQJJH01
runmqsc MQJJH01 << !
runmqsc MQCHNRDSS11 << !
ALTER QMGR +
MAXMSGL(6291456) +
CCSID(1208) +
DEADQ('MQMQJJH01.DEAD.QUEUE') +
REPOS('JJHCLUSTER') +
CHLAUTH(DISABLED) +
FORCE

ALTER CHANNEL(SYSTEM.DEF.SVRCONN) +
CHLTYPE(SVRCONN) +
MCAUSER('mqm')
END
!
runmqsc MQJJH01 << !
* lister
DEFINE LISTENER('MQJJH01LSR') +
TRPTYPE(TCP) +
PORT(1414) +
BACKLOG(0) +
CONTROL (QMGR) +
REPLACE
START LISTENER('MQJJH01LSR')
* 死信队列
DEFINE QLOCAL('MQJJH01.DEAD.QUEUE') +
DESCR(' ') +
DEFPSIST(YES) +
MAXDEPTH(999999999) +
MAXMSGL(104857600) +
REPLACE

* 集群发送通道，其中连接名是通道名中队列管理器的ip和端口
DEFINE CHANNEL('TO.MQJJH11') +
CHLTYPE(CLUSSDR) +
TRPTYPE(TCP) +
CLUSTER('JJHCLUSTER') +
CONNAME('192.168.3.53(1415)') +
DISCINT(0) +
MAXMSGL(6291456) +
REPLACE
START CHANNEL(TO.MQJJH11)
DIS CHS(TO.MQJJH11)
* 集群接受通道，其中连接名是通道名中队列管理器的ip和端口
DEFINE CHANNEL('TO.MQJJH01') +
CHLTYPE(CLUSRCVR) +
TRPTYPE(TCP) +
CLUSTER('JJHCLUSTER') +
CONNAME('192.168.3.49(1414)') +
DISCINT(0) +
MAXMSGL(6291456) +
REPLACE
START CHANNEL(TO.MQJJH01)
DIS CHS(TO.MQJJH01)

* 本地返回队列用于接受返回报文，应用或代理从这个队列中读取服务方的返回报文
DEFINE QLOCAL('LOCALQ.JJH01.RSP') +
DESCR(' ') +
DEFPSIST(NO) +
MAXDEPTH(999999999) +
MAXMSGL(6291456) +
CLUSTER('JJHCLUSTER') +
DEFBIND(NOTFIXED) +
REPLACE
END
!

MQJJH11
crtmqm MQJJH11
strmqm MQJJH11
runmqsc MQJJH11 << !
ALTER QMGR +
MAXMSGL(6291456) +
CCSID(1208) +
DEADQ('MQJJH11.DEAD.QUEUE') +
REPOS('JJHCLUSTER') +
CHLAUTH(DISABLED) +
FORCE

ALTER CHANNEL(SYSTEM.DEF.SVRCONN) +
CHLTYPE(SVRCONN) +
MCAUSER('mqm')
END
!
runmqsc MQJJH11 << !
* lister
DEFINE LISTENER('MQMQJJH11LSR') +
TRPTYPE(TCP) +
PORT(1415) +
BACKLOG(0) +
CONTROL (QMGR) +
REPLACE
START LISTENER('MQMQJJH11LSR')
* 死信队列
DEFINE QLOCAL('MQJJH11.DEAD.QUEUE') +
DESCR(' ') +
DEFPSIST(YES) +
MAXDEPTH(999999999) +
MAXMSGL(104857600) +
REPLACE

* 集群发送通道，其中连接名是通道名中队列管理器的ip和端口
DEFINE CHANNEL('TO.MQJJH01') +
CHLTYPE(CLUSSDR) +
TRPTYPE(TCP) +
CLUSTER('JJHCLUSTER') +
CONNAME('192.168.3.53(1414)') +
DISCINT(0) +
MAXMSGL(6291456) +
REPLACE
START CHANNEL(TO.MQJJH01)
DIS CHS(TO.MQJJH01)
* 集群接受通道，其中连接名是通道名中队列管理器的ip和端口
DEFINE CHANNEL('TO.MQJJH11') +
CHLTYPE(CLUSRCVR) +
TRPTYPE(TCP) +
CLUSTER('JJHCLUSTER') +
CONNAME('192.168.3.53(1415)') +
DISCINT(0) +
MAXMSGL(6291456) +
REPLACE
START CHANNEL(TO.MQJJH11)
DIS CHS(TO.MQJJH11)
* 本地请求队列，用于接受请求报文，应用或代理从这个队列中读取请求方的报文
DEFINE QLOCAL('LOCALQ.JJH01.REQ') +
DESCR(' ') +
MAXDEPTH(999999999) +
MAXMSGL(6291456) +
CLUSTER('JJHCLUSTER') +
DEFBIND(NOTFIXED) +
REPLACE
END
!

## 易达交易系统介绍



易达期货交易系统（简称易达，或ydServer）由上海汉霖信息技术有限公司设计和开发的纯软件系统。



基于我们对当代CPU体系结构、Linux操作系统、网络通讯、HFT和测量技术的逐步理解，ydServer目前可以提供微秒级别的快速期货期权交易。



易达交易系统是一套快速交易系统，主要为程序化高频交易客户服务。



目前，大多数头部期货公司都有部署易达交易系统；具体可以咨询你所开户的期货公司。



更多了解易达交易系统，可以参考[《ydServer产品简介》](https://www.hanlinit.com/products/ydserver/)。



## 易达API介绍



我们推荐投资者使用Linux开发基于ydApi的应用程序（简称客户端），当然，我们也提供Windows版本的ydApi。



我们会不定期在官网发布最新的api版本，请自行[下载](https://www.hanlinit.com/downloads/)。
    注意，使用最新版本api之前，请联系期货公司确认是否兼容。



有关易达API的更详细说明，请参考文档[《客户端应用编程接口》](https://www.hanlinit.com/docs/programming-guide/)。



如果你是易达API的初次接触者，请参考文档[《API快速入门》](https://www.hanlinit.com/docs/quick-start/)。



有关API的错误码说明，请参考文档[《易达和各交易所错误码》](https://www.hanlinit.com/docs/error-codes/)。



YDApi类具有极高的性能，但需要用户自行维护钱仓。你想了解易达是如何进行资金计算的，请参考文档[《资金计算规则》](https://www.hanlinit.com/docs/capital-calc-rules/)。



易达提供互联网的测试开发环境，具体请参考文档[《投资者互联网开发环境》](https://www.hanlinit.com/docs/dev-environments/)。




## 易达裸协议接口介绍



从1.52.0.0版本开始，易达交易系统开始支持裸协议接口，投资者可以通过直接发送自组UDP报单的方式进行交易。



更多的有关裸协议说明，请参考文档[《裸协议接口》](https://www.hanlinit.com/docs/bare-protocol/)



## CTP应用直连组件介绍



如果你以前是基于CTP接口开发的应用，为了不修改或者少修改程序，直接迁移到易达交易系统。易达推出了符合CTP API接口协议的应用直连组件。Linux平台和Windows平台均有支持。



更多说明，请参考文档[《CTP应用直连组件》](https://www.hanlinit.com/docs/ydctp/)



## ydClient介绍



ydClient是在Windows环境下使用ydApi开发的客户端应用程序，可以实现交易终端和风控终端双重目的。



投资者在开发测试阶段可用于核对交易和钱仓结果，生产阶段可用于监控程序运行情况。



有关ydClient的使用说明，请参考文档[《ydClient用户手册》](https://www.hanlinit.com/docs/ydclient/)



## 大商所组合保证金工具介绍



对于大商所盘中组合持仓，我们提供了一个自动组合工具，投资者可以从[这里](https://www.hanlinit.com/downloads/)下载。



该工具会每隔一段时间（默认是10秒），刷新用户的持仓，自动进行组合，释放部分保证金。



## 常见问题



使用过程中有任何问题，请先查询文档[《常见问题》](https://www.hanlinit.com/docs/common-questions-2/)。


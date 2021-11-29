引言
==

易达期货交易系统（简称易达，或ydServer）运行在Linux服务器上，投资者使用易达客户端应用程序接口（简称ydApi）与ydServer快速通信。

ydServer体系结构如下：

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydServer_arch.png)

ydClient是在Windows环境下使用ydApi开发的客户端应用程序，可以实现交易终端和风控终端双重目的。

*   期货公司客户可以使用该界面用交易账户和密码在应急情况下的报撤单
*   期货公司的管理员和风控人员使用该界面可以在应急情况下为普通用户撤单，设置交易权限，实施出入金等资金管理，查看使用ydServer的普通用户的账户交易情况和风险度。

ydClient的文件
===========

ydClient无需安装，只要将下列文件拷贝到目标机器的Windows环境下即可运行。

**目录或者文件名**

**说明**

ydClient.exe

风控终端主程序。

yd.dll

ydApi的Windows DLL封装链接库。

ydClient.ini

风控终端运行的配置程序，主要定义ydServer的通信参数。

log

ydClient.exe运行中生成日志使用的目录，每日一个文件，txt格式。

mdList.txt

ydClient从ydServer订阅行情的合约列表。

YDClient.ini
------------

    ###########################################
    ######### Network configurations  #########
    ###########################################
    
    # IP address of yd trading server
    TradingServerIP=127.0.0.1
    
    # TCP port of yd trading server, other ports will be delivered after logged in
    TradingServerPort=51000
    
    ###########################################
    ######### Trading configurations  #########
    ###########################################
    
    # Whether using UDP when sending orders, no for sending TCP orders
    # When accessing yd trading server via Internet or other UDP-banned network enviornments, must set to no
    UDPTrading=no
    
    # Timeout of select() when receiving order/trade notifications, in millisec. -1 indicates running without select()
    TradingServerTimeout=10
    
    # Affinity CPU ID for thread to receive order/trade notifications, -1 indicate no need to set CPU affinity
    TCPTradingCPUID=-1
    
    ###########################################
    ####### MarketData configurations  ########
    ###########################################
    
    # Whether need to connect to TCP market data server
    ConnectTCPMarketData=yes
    
    # Timeout of select() when receiving TCP market data, in millisec. -1 indicates running without select()
    TCPMarketDataTimeout=10
    
    # Affinity CPU ID for thread to receive TCP market data, -1 indicate no need to set CPU affinity
    TCPMarketDataCPUID=-1
    
    # Whether need to receive UDP multicast market data
    ReceiveUDPMarketData=no

主界面
===

登录界面
----

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_login01.png)

如上图，点击运行ydClient后将出现登录界面。输入用户名和口令即可。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_login_in.png)

登录后的主界面里面分成了六个不同区域：

*   账户资金汇总信息窗口：如果管理员登录，则显示交易账户资金汇总信息，包括账户名称，风险度、可用资金、余额和出入金信息的汇总；  
    如果某个交易账户的风险度增加达到ydServer/config/Client.csv指定的阀值，其颜色会发生变化以起到提醒作用。  
    交易账户登录的话本窗口显示合约订阅行情。
*   合约订阅行情显示窗口：显示订阅合约的最新价、一档的买入价量和卖出价量、成交量和发生的行情时间。
*   账户资金明细窗口：显示指定（如果管理员登录并在交易账户资金汇总信息窗口中点击选择了账户）账户或者自己（交易账户）的资金明细情况。
*   账户持仓明细窗口：显示指定账户（管理员登录）或者自己（交易账户）的持仓明细情况：  
    包括冻结开仓量、冻结平仓量、合约最新价、保证金和持仓盈亏。
*   账户报单明细窗口：显示指定账户（管理员登录）或者自己（交易账户）的目前在挂的有效报单明细情况：  
    包括买卖、开平、投保情况，申报价格和数量；报单是普通还是强平；报单编号；  
    报单所使用的席位情况；报单的成交量等信息。
*   账户询价明细窗口：显示指定账户（管理员登录）或者自己（交易账户）的目前在挂的有效询价单明细情况：  
    包括买开、买投、买价、买量、卖开、卖投、卖价、卖量；  
    报单编号，买单号，卖单号，报单所使用的席位情况；询价号等信息。
*   命令窗口：可以通过命令窗口修改口令、查询账户明细、系统业务（如各类流控信息）参数、  
    费率信息（保证金率和手续费率）、交易权限和出入金功能（仅管理员可用）。

交易账户行情显示
--------

### 订阅/退订行情

在行情显示窗口的空白处鼠标左键双击会弹出【选择订阅行情的品种】，比如希望增加cu1906和cu1907的期权的报价信息：

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_md.png)

选择合约后鼠标左键双击即可显示新加入合约的行情。退订行情可以在指定合约行鼠标右键双击即可。

选择后的合约信息会记录在ydClient运行目录的mdList.txt文件中，退出后重新登录将无需再次选择合约。

### 报单

在指定合约显示行的买入价/买入量上双击鼠标左键，即可弹出卖出报单窗口；

如果在卖出价/卖出量上双击鼠标左键，即可弹出买入报单窗口。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_orde_input.png)

投保标志、报单类型、价格和数量、指定席位下单，以及报单种类可以手工修改。

*   投保标志：包括投机、套利（仅CFFEX使用）和套保。
*   类型：包括普通报单、FAK报单、FOK报单和市价报单。
*   连接：包括任意、指定、倾向。
*   编号：席位编号
*   种类：包括报单、询价

在价格和数量字段右侧的两列数字分别代表该合约最新的买入卖出最新的价量，会随着行情改变而改变。

点击【报单】即可向系统发送报单请求。【放弃】则回到主界面。

本例中以280的价格开仓买入1手投机仓位的au1906报单。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_orde_main01.png)

报单成功后在持仓明细窗口显示了所冻结的信息：冻结的开仓量1手；开仓量1手；冻结保证金16800 。因为尚未成交，因此持仓盈亏为0。

在挂有效报单窗口中显示了报单的情况：状态为在挂，当前成交量为0；报单的本地编号为0，柜台系统给出的系统编号为13，报单时刻为21:40:29。

在持仓明细窗口中鼠标左键双击也可以弹出报单窗口用于快速平仓：其买卖方向、开平仓标志、投保标志已经设置为仓位的相反值。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_orde_close.png)

### 撤单

在挂有效报单窗口中双标左键双击报单会弹出撤单窗口，并询问是否确认（本例中双击了第13号报单）：

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_orde_cancel01.png)

确认后该报单的撤单请求会送往服务器。

如果撤单成功在挂有效报单窗口中的报单会被删除，相应的冻结资金和开仓量会被撤销，但该合约的撤单数会增加。

我们可以在【账户明细】窗口中查询到撤销的报单：

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_orde_cancel_dtl01.png)

### 询价报单

在指定合约上双击鼠标左键，即可弹出卖出询价报单窗口。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_orde_action.png)

### 询价撤单

在挂有效询价单窗口中鼠标左键双击询价单会弹出撤单窗口，并询问是否确认（本例中双击了第1号询价单）：

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_orde_action_cancel.png)

交易账户资金明细
--------

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_account_fund01.png)

上左图红线部分圈出了066交易账户的资金使用明细情况：

*   余额 = （昨余额 + 入金 – 出金 – 冻结出金）\* 使用限度，目前限度设置为100%，因此昨余额全部可用。  
    如果该账户在多家交易所的多柜台系统使用，需要分配使用限度。总限度不得超过100%。
*   可用 = 余额 + 平仓盈亏（盈为正，亏为负）– 手续费 + 权利金 – 保证金 - 持仓亏损 。
*   保证金为持仓明细中的保证金之和。
*   持仓盈亏会随着该客户的持仓以及最新行情实时计算并显示。等于持仓明细中的【仓盈】之和。  
    持仓盈亏为负代表亏损，需要从可用中减去；为正代表盈利，但不能用于新开仓，因此可用中并不增加。
*   交易权限：管理员可以在盘中修改交易账户是否可以交易、只可平仓或者不能交易。  
    即使不能交易，该账户也可进行撤单操作，但不能开仓或者平仓。
*   风险度 = 保证金 / 权益 \* 100%。  
    如果该值超过ydServer/config/Client.csv中规定的报警值则会显示成橙色（第一档WarningLevel值）或红色（第二档WarningLevel值），起到提示作用。

交易账户持仓明细
--------

该窗口显示交易账户的现有持仓明细信息。

此信息可以在【账户明细】命令显示的持仓明细窗口中保留：

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_account_positions.png)

“类型”字段：如果在多空之前有今，代表的是当日新开仓位。

“持仓”字段表示当前仓位；

“昨仓”表示代表历史仓位；

“冻开”表示当前在挂报单的开仓数量；

“冻平”表示当前在挂报单的平仓数量；

“开量”表示今日开仓成交量；

“价格”为持仓均价；

“保证金”为冻结保证金金额；

“仓盈”为根据最新对手价计算的持仓盈亏；

“报单仓”为根据交易所的报单回报中的成交量所计算出的持仓量，此值理应等于“持仓”字段 。

交易账户在挂报单明细
----------

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_orde_dtl.png)

“类型”字段为普通限价报单、FAK报单、FOK报单或者市价报单。

“实连”字段表示指定了哪个席位报单。

“系统号”字段表示ydServer的报单编号。只有有效报单才有报单编号，否则为-1。

“状态”字段表示报单状态：在挂报单已被交易所确认且有效，可能部分成交，也可能未成交；

*   撤单表示报单已被撤销，可能是客户主动撤销，也可能是FAK、FOK类或市价类报单被交易所系统自动撤销
*   全成代表完全成交
*   拒绝表示报单被ydServer或者交易所拒绝，在账户明细命令中的报单明细窗口中会显示是交易所拒绝还是ydServer拒绝，以及错误编号。

如下图所示，该窗口增加了每手报单的所需的保证金和权利金；该窗口还显示了当日已挂的历史报单和错误报单信息。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_accountdtl_orde.png)

交易账户在挂询价单明细
-----------

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_ordeaction_dtl.png)

注意：一条询价单自动派生出两条报单；派生的报单不允许撤单，询价挂单可以撤销。

账户明细
====

点击【账户明细】按键后弹出如下明细窗口。

主要包括6方面的信息：报单明细窗口；成交明细窗口；流控信息窗口；持仓明细窗口；产品明细；询价单明细窗口。

其中报单明细窗口已经在【3.5 交易账户在挂报单明细】中作出说明，持仓明细窗口在【3.4 交易账户持仓明细】中作出说明。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_accountdtl_main01.png)

成交明细
----

报单在申报时仅冻结保证金（如果开仓）和持仓（如果平仓），并未计算手续费。

成交后则根据设置的手续费率计算并显示。

这里的手续费仅仅是试算提示作用，具体金额需要期货公司结算系统根据全天的成交确定。

流控明细
----

*   “撤单数”字段：表示限价报单使用的撤单笔数。
    
    客户报入一笔限价报单时，如果校验成功，ydServer会将撤单数加1；  
    如果交易所拒绝该笔报单，ydServer会根据回报将撤单数减1；  
    如果该笔报单全部成交，ydServer会根据最后一笔成交回报或者报单回报将撤单数减1。  
    但如果报单在挂状态，则维持此撤单数，因为客户可能在最后时刻撤单。
*   “可成量”字段：表示所有在挂报单和已经成交（或部分成交并撤单）报单的可能成交手数。
    
    如果目前没有在挂报单，该字段等于“成交量”字段。
*   “可开量”字段：表示所有在挂开仓报单和已经成交（或部分成交并撤单）开仓报单的可能成交手数。
    
    如果目前没有在挂报单，该字段等于对应持仓明细记录中国的“持仓”字段。

产品明细
----

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_accountdtl_proddtl.png)

“持仓”字段表示当前仓位；

“冻开”字段表示当前在挂报单的开仓手数。

“撤单数”字段表示当前在挂报单笔数和已经撤单笔数之和。

“成交量”字段表示当日成交手数。

“可成量”字段表示所有在挂报单和已经成交（或部分成交并撤单）报单的可能成交量。

“可开量”字段表示所有在挂开仓报单和已经成交（或部分成交并撤单）开仓报单的可能成交手数。

“保证金”字段表示当前使用的保证金额。

“多仓”字段表示多头持仓手数。

“多保金”字段表示当前多头持仓使用的保证金额。

“空仓”字段表示空头持仓手数。

“空保金”字段表示当前空头持仓使用的保证金额。

持仓的组合和拆分
--------

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_accountdtl_cp01.png)

双击“持仓明细窗口”可以调出持仓的组合和拆分窗口。

如果有现有组合持仓，则双击可以进行组合拆分；

如果有可能组合持仓，则可以进行持仓组合。

系统参数
====

点击【系统参数】按键后弹出如下窗口，

主要包括7方面的信息：交易所信息；资金计算参数；产品基本信息；合约基本信息；组合持仓信息；以及产品和合约限仓信息。

本文中的限制类参数仅做示范使用，具体要由期货公司按照交易所要求和客户风险自行设置。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_sysparams_main01.png)

交易所信息
-----

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_sysparams_exchange.png)

“席位数”字段表示当前ydServer连接该交易所使用的席位数量，各席位编号从0开始依次编号。本例中使用了两个席位。

“用今仓”字段代表是否需要在平仓时明确指示历史仓或者今日新仓，仅SHFE需要此配置。本例中使用今仓。

“套利户”字段代表是否支持套利账户，仅CFFEX需要此配置。

“先平今”字段代表平仓时优先使用今日新仓还是历史仓位。

“单大边”字段代表是否按产品多空方向计算保证金后收取大头。

双击“交易所信息”窗口，可以调出“交易所席位优选连接”设置。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_sysparams_xwselect.png)

资金计算参数
------

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_sysparams_fund.png)

本例中对期货开仓保证金以开仓价来计算，对于期权开仓保证金以max（昨结算价，最新价）来计算；

对于期货委托以报单价来冻结保证金，对于期权卖方委托以昨结算价来冻结保证金。

产品基本信息
------

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_sysparams_prodbase.png)

“基础乘数”，如果是期货期权合约，表示每手期权对应的期货合约数量，否则无意义。

合约基本信息
------

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_sysparams_instrubase01.png)

产品限仓信息
------

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_sysparams_prodlimit.png)

每个交易账户在对应产品的投机/套保维度上不能超过以下限制（设为-1则不限制）

系统使用了“悲观”算法进行流量等控制，  
比如对于撤单笔数，如果客户申报了限价单，则其撤单笔数立即增加；  
如果该报单全部成交或者被交易所拒绝，则其撤单笔数会返还。  
该规则同样使用于开量、限仓和限制成交手数。

合约限仓信息
------

每个交易账户在对应合约的投机/套保维度上不能超过以下限制（设为-1则不限制），内容同前节，不再赘述。

组合持仓信息
------

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_sysparams_cpinfo.png)

该窗口显示系统支持的组合持仓类型，优先级等信息。

费率信息
====

点击【费率信息】按键后弹出各合约在投机/套利/套保维度上的保证金率（多头空头分别列出）和手续费率（开仓、平仓和平今仓分别列出，还包括报单手续费和撤单手续费）。

图中两个连续%%，代表万分之几。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_fee_futures01.png)

上图中ag1904投机和套保的单位保证金率为7%，  
即每手保证金等于成交金额或者报单金额 \* 合约乘数 \* 0.07。  
如果没有%，只有数字，则按照成交或者报单手数来收保证金，每手保证金等于该数字。  
如果既有%，又有数字，则表示上述两种保证金的和。

手续费也是按照同样方式计算。

上例中报单手续费和撤单手续费统一设置为0。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_fee_options01.png)

上述费率为交易前从期货公司主交易系统的结算数据库中获取，交易当天无法修改。

口令修改和交易权限
=========

口令修改
----

交易账户能够修改自己的口令。

管理员可以设置交易账户的口令；管理员可以修改自己的口令，但不能修改其他管理员的口令。

交易权限
----

交易账户能够查询自己的交易权限。管理员可以设置交易账户的交易权限。

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_traderights.png)

管理员调整资金
=======

【调整资金】只能由管理员使用。点击【调整资金】进入以下界面：

![](https://www.hanlinit.com/wp-content/uploads/2021/01/ydClient_fundchange.png)

选择账户
----

如上图所示，在蓝色的下拉框中选择不同的账户，左边窗口会显示该账户的资金明细情况。

调整资金使用限度
--------

资金使用限度介于0至1之间。表示在ydServer中交易账户的可用资金。

在日初时：可用 = 昨余额 \* 使用限度。

调整使用限度后“可用”也随之发生变化，相应的，“风险度”也发生变化。

出入金
---

期货公司在主交易系统入金后即可在本系统中出入金。

出入金数额的使用限度部分立即减少或者增加到当日“可用”资金中。

注意，出入金数额为出入金业务中的确认额，无需和使用限度相乘，系统会自动做此系数的乘法。
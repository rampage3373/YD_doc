Overview
========

This document mainly walks through the methods of applying YD's internal API. Besides YD's original API, we also provide raw protocol interface and CTP simulation interface. For more information about raw protocol interface, see raw protocol interface. For more information about CTP simulation interface, see direct connection package.

ydApi is available on Linux and Windows. The Linux version of ydApi has plenty of optimizations based on the Linux platform, making its performance more outstanding than the Windows version. Therefore, we suggest all users use Linux version ydClient.

The communication between ydClient and ydServer mainly consisted of upstream and downstream. In upstream, requests (like system login, placement and cancellation of orders, changing password and uploading heartbeat messages) are transferred from ydClient to ydServer. In downstream, data are sent from ydServer to ydClient. (i.e. initializing static data, responses of placed/filled orders and market data) Moreover, initializing static data at the beginning of the day will also sync to the local database. Initialization/orders/order return/market data will execute ydClient's "call-after" function. In ydClient, you can use local call of YDApi::getX series of functions to obtain initialized static data of the beginning of the day. However, users need to calculate their margin, cost of the transaction, available capital and position according to orders/matched trade returns.

![](https://www.hanlinit.com/wp-content/uploads/2021/10/24d18c999ad9ee8e540a5c34fed4a64.png)

Note: ydApi indicates the Hanlin's api itself and YDAPI is the class of ydApi.

ydClient Release
================

ydClient Release contains:

*   ydAPI
    *   include: ydApi's header file
    *   linux64: ydApi's DLL(Dynamic-link library) for Linux OS
    *   win32: ydApi's DLL for Windows OS
    *   example: examples of using ydApi to develop
*   ydClientWin
*   ydCTP
    *   include: CTP's header file
    *   linux64: ydCTP's DLL for Linux OS
    *   win32: ydCTP's DLL for Windows OS
    *   example: example of using ydCTP to develop

Opreating System
----------------

According to regulatory requirements, ydApi need to collect data from hardware's BIOS detail. It might require administrator permission (Windows OS) or root permission (Linux OS). Besides that, ydApi doesn't have any operating system dependency in Windows.

Linux:

*   Kernel: 2.6.18 or later
*   GNU libc: 2.5 or later ( you can use "ldd --versions" to check)
*   Applying dmidecode allows general users to retrieve hardware's BIOS detail. To facilitate your strategy's security level, you can adjust the method of runing dimecode by ReportMethod in the config.txt file. In Linux, 0 indicates suid and 1 indicates sudo.

Initialization
==============

General steps
-------------

ydServer only provides data initialization mode under push mode, which does not offer any searching function. Data initialization based on push mode mainly includes the following steps:

1.  Start ydClient.exe, calling makeYDApi or makeExtendedApi to create Api object. The configuration file needs to be set up to generate Api by providing parameters.
2.  ydClient.exe calls start function to start ydApi, YDListener needs to be implanted when starting.
3.  ydApi will automatically try to connect to ydServer. If the connection succeeds, that YDListener::notifyReadyForLogin will be called.
4.  In YDListener::notifyReadyForLogin class, ydApi will call the login function which requires client ID, password, AppID and AppAuthCode.
5.  After logging in successfully, ydApi will call YDListener::notifyLogin, and the message returned will contain the user's maximum reference ID, which covers past (before today) static data at the beginning of the day and past established/filled orders.
6.  After logging in successfully, ydServer will push today's static data at the beginning ( includes capital, position, margin rate and commission rate). When finished, ydApi will call YDListener::notifyFinishInit to notify users that retrievement is over. Note that trading orders submitted and filled earlier today have been retrieved yet. Any move asking for information about position or capital will fail.
7.  ydServer will push data of today's trading orders to ydClient through YDListener::notifyOrder and YDListener::notifyTrade. When the reference number of pushed data equals the maximum reference number, ydApi triggers YDListener::notifyCaughtUp to notify that trading data are up to date.
8.  So far, all data have been initialized, and users can start handling their own business. If there are new trading orders, ydApi will call YDListener::notifyOrder and YDListener::notifyTrade.

ydClient reconnection steps
---------------------------

1.  When ydApi detects disconnection, it will try reconnecting to ydServer. If reconnection succeeds, ydApi will send the maximum reference ID, retrieved before disconnection, to ydServer and call YDListener::notifyReadyForLogin.
2.  In YDListener::notifyReadyForLogin class, ydApi will call the login function which requires client ID, password, AppID and AppAuthCode.
3.  After logging in successfully, ydServer will push trading data after disconnection through YDListener::notifyOrder and YDListener::notifyTrade. When the reference number of pushed data equals the maximum reference number before disconnection, ydApi triggers YDListener::notifyCaughtUp to notify that trading data are up to date.
4.  The system is back to normal.

Client Configuration File
=========================

When the ydClient calling method makeYDApi, the configuration file used by ydClient needs to be assigned. The configuration file mainly includes three categories:

*   Network configurations include ydServer's IP address and port. In that, the port is the TCP server port provided by your futures company. ydServer will deliver other ports after login. Whether to use UDP for sending orders is controlled by UDPTrading.
*   Trading configurations include UDPTrading and TradingServerTimeout.
*   Market data configurations include either retrieving ydServer's market data by TCP or UDP.

The configuration file list below is the best example template for ydAp in a developing environment. This template achieves:

*   Using UDP to send orders.
*   The busy mode in receiving TCP order return, and set TCP Trading CPU ID to 3.
*   It is not connecting to TCP market data or UDP market data.

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
    UDPTrading=yes
    
    # Timeout of select() when receiving order/trade notifications, in millisec. -1 indicates running without select()
    TradingServerTimeout=-1
    
    # Affinity CPU ID for thread to receive order/trade notifications, -1 indicate no need to set CPU affinity
    TCPTradingCPUID=3
    
    ###########################################
    ####### MarketData configurations  ########
    ###########################################
    
    # Whether need to connect to TCP market data server
    ConnectTCPMarketData=no
    
    # Timeout of select() when receiving TCP market data, in millisec. -1 indicates running without select()
    TCPMarketDataTimeout=-1
    
    # Affinity CPU ID for thread to receive TCP market data, -1 indicate no need to set CPU affinity
    TCPMarketDataCPUID=-1
    
    # Whether need to receive UDP multicast market data
    ReceiveUDPMarketData=no
    
    

Network Configurations
----------------------

Network configurations include YD Trading Server Port and ID address. For more information about network configuration in the development environment, please contact the IT Department of your futures company.

**RecorySiteCount:** Number of the recovery site. It is used to achieve high availability at the expense of a bit of performance of order notification. 0 for no recovery, 1 for recovery always use the primary site, 2 for recovery use primary and secondary sites. If the primary site crashes, the secondary site (Use TradingServerIP2 and TradingServerPort2 to configure) will ensure that ydApi is reconnected to the trading server and recovered to the latest state.

**TradingServerIP:** IP address of the primary trading server.

**TradingServerPort:** Port of the trading server. ydServer generally provides three ports: TCP Trading Server Port, TCP Market Data Server Port and UDP Trading Server Port. (UDP Market Data Server Port is closed by default). The required port number is the TCP port number of the primary trading server. ydServer will deliver other ports after logged in. Set the "UDPTrading " value to "yes" if you wish to use UPD instead of TCP. Note that directly setting the "TradingServerPort" value into a UDP trading server port number will cause failure to connect.

**TCPMarketDataServerPort:** Port of TCP market data server. Try not to use or change this parameter since it is automatically delivered by ydServer. If NAT(Network Address Translation) is applied while accessing ydServer via Internet, TCP market data server port would be replaced and cannot receive TCP market data. In this case, the parameter needs to be filled with the port number provided by the futures company.

**UDPTradingServerPort:** Port of UDP trading server. Try not to use or change this parameter since it is automatically delivered by ydServer. If NAT(Network Address Translation) is applied while accessing ydServer via the Internet, the UDS trading server port would be replaced and not receive UDP trading data. In this case, the parameter needs to be filled with the port number provided by the futures company.

**TradingServerIP2:** IP address of the secondary trading server. Valid only when RecoverySiteCount equals 2.

**TradingServerPort2:** TCP port of the secondary trading server. Valid only when RecoverySiteCount equals 2.

**TCPMarketDataServerPort2:** TCP port of the secondary market data server. Valid only when RecoverySiteCount equals 2.

**UDPTradingServerPort2:** UDP port of the secondary trading server. Valid only when RecoverySiteCount equals 2.

Trading Configurations
----------------------

**RecalcMode:**Recalculation mode for recalculation of margin and position profit. Valid when using ydExtendedApi. Set as "auto" by default.

*   auto: Subscribe market data and automatcially recalculate in proper time. Use RecalcMarginPositionProfitGap and RecalcFreeGap to contorl the gap between recalculations and delay of recalculations.
*   subscribeOnly: Subscribe specific market data, and recalcMarginAndPositionProfit should be called explicitly by users.
*   off: No recalculation. Generally, users should not use this mode.

**RecalcMarginPositionProfitGap:** Gap between recalculations, in milliseconds. It is set as 1000 by default. It will be adjusted to 1000 if less than 1000. Valid when RecalcMode is set to auto.

**RecalcFreeGap:** Delay of recalculation after market data arrives to avoid collision with input order, in milliseconds. Users can adjust this parameter, so there's enough time to recalculate according to different futures Exchanges' conditions. Users should set it between 0 and 100. It will be adjusted to 100 if larger than 100 and will be adjusted to 0 if smaller than 0. Valid when RecalcMode is set to auto.

**UDPTrading:** Whether using UDP or TCP when sending orders. "yes" means sending UDP orders, and "no" means sending TCP orders. We highly recommend using UDP for sending orders in the development environment because ydServer's excellent performance is based on the UDP trading. TCP can not support ydServer to reach its best performance with TCP's relatively slow property. However, UDP trading isn't available in several situations like internet or firewall secured network without special network configurations. When accessing ydServer via the Internet or other UDP-banned network environment, must set "UDPTrading" to "no". Otherwise, you won't be able to send orders, although you can log in.

**TradingServerTimeout:** Timeout of select() when receiving order/trade notifications, in milliseconds. -1 indicates running without select() and will maximize CPU usage to 100%. Suppose the affinity of threads and CPU/Core has not been set up to distribute the usage of CPU/Core wisely. In that case, it will severely slow ydClient's performance, and delay will increase significantly.

**TCPTradingCPUID:** Affinity CPU ID for the thread to receive order/trade notification, -1 indicated no need to set CPU affinity.

MarketData Configurations
-------------------------

Please note, in ydServer, neither TCP market data nor UDP multicast market data is not high-speed market data. By far, it is only used for calculating margin and position profit by ydServer and ydClient. If you need market data in a development environment, please contact your futures company.

**ConnectTCPMarketData:** Whether connect to TCP market data server or not.

**TCPMarketDataTimeout:** Timeout of select() when receiving TCP market data, in milliseconds. -1 indicates running without select() and will maximize CPU usage to 100%. If ydClient has not been set up to distribute, and delay will increase significantly.

**TCPMarketDataCPUID:** Affinity CPU ID for the thread to receive TCP market data, -1 indicate no need to set CPU affinity.

**ReceiveUDPMarketData:** Whether receive UDP multicast market data or not. By far, all ydServer's UDP multicast market data features are closed. Please set it to "no".

Other Configurations
--------------------

**ReportMethod:** Method of calling dmidecode when generating ydClient report. Not available in Windows. In Linux, 0 indicates using suid, which uses chmod +s/usr/sbin/dmidecode to gain permission; 1 indicates using sudo that ydApi will gain hardware's detail through sudo dmidecode.

**AppID and AuthCode:** Users can set up AppID and AuthCode in the configuration file. If you type NULL when login, ydApi will use values in the configuration file.

**LogDir:** Destinated directory of your log report. Log report won't be created by default. If the corresponding directory doesn't exist, ydClient won't create the log report either.

High Availabilities
===================

In ydClient 1.108, we provide two high availabilities(HA): Single-Server HA and Active/Standby Server HA.

When high availability is on, ydAPI will search for available nodes and connect to them if the primary ydServer loses connectivity, as long as the new ydServer has the same signature as the old server.

You can configure [RecoverySiteCount](#RecoverySiteCount) and other parameters to turn on ydAPI's high availabilities(HA). [Click here for more details](#RecoverySiteCount).

You don't need to configure anything to use Single-Server HA. Upon starting, ydApi won't quit when ydServer restarts. Under "RecoverySiteCount=1", when ydServer restarts, ydApi will first receive message "YD\_AE\_ServerRestarted=4" and then automatically reconnect to ydServer.

Active/Standby Server HA can ensure the greatest high availability: When downtime, network disconnection, or other malfunctions occur, the standby server will restore users' trading in the shortest time. Active/Standby Server HA makes three servers into a group:

*   The primary server works in normal condition.
*   The secondary server usually stays in an inactivated state and only works when a failover occurs.
*   The arbitration server checks the server's availability (without performance restriction) and decides whether to switch or not.

To avoid useless failover, we generally suggest that futures companies use two networks. To configure active/standby server HA, you need a prepaid authorization file to activate. When ydApi uses Active/Standy Server HA (RecoverySiteCount=2), it will connect to the primary server. If the primary server cannot work(downtime, network disconnection), ydApi will automatically connect to the secondary server. Active/Standy Server HA's connection system is shown below:

![](https://www.hanlinit.com/wp-content/uploads/2021/10/749fb39e84a04eaab83b9dfa0c2750e.png)

No matter what HA is in use, all trading data are saved. However, if Exchanges return data during a breakdown, the system might not recognise and match the user's order. Thus, order returns and matched trade returns with OrderRef and OrderLocalID equal -1 might be received. There are two solutions, and you should handle that data based on conditions:

*   Directly delete the pending orders after YD\_AE\_ServerRestarted received, then handle those order returns and matched trade returns with OrderRef and OrderLocalID equals -1 as orders and trading from the external system.
*   Match those pending orders with order returns according to their contract, price, volume and other properties. If there are orders left unmatched, it means that they have not been sent to Exchanges. Just delete those unmatched pending orders locally.
*   HA ensures data's entirety when failover occurs. If the server restarts and you restart ydApi, the server will set orders before restart's OrderRef and OrderLocalID as -1.

Data Types
==========

All data types are located at file ydDataType.h.

ProductClass
------------

Types of products.

**Definition**

**Description**

YD\_PC\_Futures=1

Future

YD\_PC\_Options=2

Options

YD\_PC\_Combination=3

Combination

YD\_PC\_Index

Index

YD\_PC\_Cash=5

Cash

OptionsType
-----------

Types of options.

**Definition**

**Description**

YD\_OT\_NotOption=0

Not an option

YD\_OT\_CallOption=1

Call option

YD\_OT\_PutOption=2

Put option

Direction
---------

Trading directions of an order or orders.

**Definition**

**Description**

YD\_D\_Buy=0

Long

YD\_D\_Sell=1

Short

YD\_D\_Make=0

Make a combination order. Valid when "YDOrderFlag" equals "YD\_YOF\_CombPosition".

YD\_D\_Split=1

Split a combination order. Valid when "YDOrderFlag" equals "YD\_YOF\_CombPosition".

PositionDirection
-----------------

Long or short position.

**Definition**

**Description**

YD\_PD\_Long=2

Long position

YD\_PD\_Short=3

Short Position

PositionDate
------------

Trading date of position.

**Definition**

**Description**

YD\_PSD\_Today=1

Position established today. (Available for SHFE & INE)

YD\_PSD\_History=2

**Position established before. Historical position.**

HedgeFlag
---------

Types of trading strategies.

**Definition**

**Description**

YD\_HF\_Speculation=1

Speculation (CFFEX only)

YD\_HF\_Arbitrage=2

Arbitrage

YD\_HF\_Hedge=3

Hedge

YD\_HF\_Internal=4

For internal use.

YD\_MaxHedgeFlag=4

For internal use.

In our system, HedgeFlag's rules of use is list below:

*   CFFEX: supports YD\_HF\_Speculation, YD\_HF\_Arbitrage and YD\_HF\_Hegde
*   SHFE, INE & DCE: support YD\_HF\_Speculation and YD\_HF\_Hegde

TradeRight
----------

Trading permission for users.

**Definition**

**Description**

YD\_TR\_Allow=0

Allow users to open positions, close positions and cancel orders.

YD\_TR\_CloseOnly=1

Allow users to close positions and cancel orders.

YD\_TR\_Forbidden=2

Only allow users to cancel orders.

OffsetFlag
----------

Opening or liquidation indicator.

**Definition**

**Description**

YD\_OF\_Open=0

Opening positions

YD\_OF\_Close=1

Liquidating positions. (Not available for SHFE & INE)

YD\_OF\_ForceClose=2

Forced Liquidation.

YD\_OF\_CloseToday=3

Liquidating today's new positions.(Available for SHFE & INE)

YD\_OF\_CloseYesterday=4

Liquidating historical positions. (Available for SHFE & INE)

Based on our understanding: SHFE and INE differentiate today's positions' liquidation and historical positions' liquidation. When liquidating, CFFEX prefers to use today's positions first; DCE & CZCE prefers to use historical positions first. You can find out which position is used during liquidating by "YDExchange::UseTodayPosition".

Users need to calculate historical and today's position according to the initial position and daily volume and choose different indicators according to different exchanges.

In our system, OffsetFlag:

*   SHFE & INE: support YD\_OF\_CloseToday and YD\_OF\_CloseYesterday. If you place a liquidating order without YD\_OF\_CloseToday indicator, exchanges will automatically bind the order with YD\_OF\_CloseYesterday.
*   CFFEX & DCE: support YD\_OF\_Close. If you place an order without YD\_OF\_Open indicator, exchanges will automatically bind the order with YD\_OF\_Close.

OrderType
---------

Types of trading orders.

**Definition**

**Description**

YD\_ODT\_Limit=0

Limit Order

YD\_ODT\_FAK=1

Fill And Kill (FAK) Order

YD\_ODT\_Market=2

Market (MKT) Order

YD\_ODT\_FOK=3

Fill Or Kill (FOK) Order

Trading orders, which are defined in the YD System, are the combinations of exchanges' orders. The comparison table of YD orders and exchanges' orders are listed below:

**Exchanges**

**YD\_ODT\_Limit**

**YD\_ODT\_FAK**

**YD\_ODT\_Market**

**YD\_ODT\_FOK**

CFFEX

FFEX\_FTDC\_TC\_GFD  
FFEX\_FTDC\_OPT\_LimitPrice  
FFEX\_FTDC\_VC\_AV

FFEX\_FTDC\_TC\_IOC  
FFEX\_FTDC\_OPT\_LimitPrice  
FFEX\_FTDC\_VC\_AV

FFEX\_FTDC\_TC\_IOC  
FFEX\_FTDC\_OPT\_AnyPrice  
FFEX\_FTDC\_VC\_AV

FFEX\_FTDC\_TC\_IOC  
FFEX\_FTDC\_OPT\_LimitPrice  
FFEX\_FTDC\_VC\_CV

SHFE

SHFE\_FTDC\_TC\_GFD  
SHFE\_FTDC\_OPT\_LimitPrice  
SHFE\_FTDC\_VC\_AV

SHFE\_FTDC\_TC\_IOC  
SHFE\_FTDC\_OPT\_LimitPrice  
SHFE\_FTDC\_VC\_AV

SHFE\_FTDC\_TC\_IOC  
SHFE\_FTDC\_OPT\_AnyPrice  
SHFE\_FTDC\_VC\_AV

SHFE\_FTDC\_TC\_IOC  
SHFE\_FTDC\_OPT\_LimitPrice  
SHFE\_FTDC\_VC\_CV

DCE

OT\_LO  
OA\_NONE

OT\_LO  
OA\_FAK

OT\_MO  
OA\_FAK

OT\_LO  
OA\_FOK

CZCE

FID\_OrderType=0 //Limit Order  
FIDMatchCondition=3 //Good till today

FID\_OrderType=0 //Limit Order  
FID\_MatchCondition=2 //FAK Order

FID\_OrderType=1 //Market Order  
FID\_MatchCondition=2 //FOK Order

FID\_OrderType=0 //Limit Order  
FID\_MatchCondition=1 //FAK Order

OrderStatus
-----------

Status of orders.

**Definition**

**Description**

YD\_OS\_Accepted=0

Order is accepted by ydServer and ydServer towards it to the exchange. No response has been received from the exchange yet. This status will not be sent to ydAPI and ydClient.

YD\_OS\_Queuing=1

Order is verified by the exchange. It might be traded partly or not.

YD\_OS\_Canceled=2

Order is cancelled. It may be cancelled by users or cancelled because of order types. (FAK, MKT, etc.)

YD\_OS\_AllTraded=3

Order is completetly filled.

YD\_OS\_Rejected=4

Order is rejected by ydServer or the exchange. Error code is saved in ErrorNo field of YDInputOrder. If the error code is less than 1000, it is rejected by ydServer. (Error codes are listed in ydError.h) If the error code is greater than 1000, it is rejected by the exchange. (Error code - 1000 = exchange's error code)

YDOrderFlag
-----------

Orders' indicator.

**Definition**

**Description**

YD\_YOF\_Normal=0

Standard order

YD\_YOF\_QuoteDerived=1

Order derived by quotation. Those orders cannot be directly modified.

YD\_YOF\_OptionExecute=2

Request for exercising an option

YD\_YOF\_OptionAbandonExecute=3

Request for abandoning an option execution

YD\_YOF\_RequestForQuote=4

Request for quotations from the market maker

YD\_YOF\_CombPosition=5

Combined position

YD\_YOF\_OptionExecutetogether=6

Option trading combination execution

YDQuoteFlag
-----------

Quotation indicator.

**Definition**

**Description**

YD\_YQF\_ResponseOfRFQ=1

Response of the requests for quotation

YDCombHedgeFlag
---------------

Combined trading strategies

**Definition**

**Description**

YD\_CHF\_SpecSpec=1

Speculation + Speculation

YD\_CHF\_SpecHedge=2

Speculation + Hegde

YD\_CHF\_HedgeHedge=3

Hedge + Hedge

YD\_CHF\_HedgeSpec=4

Hegde + Speculation

YD\_MaxCombHedgeFlag=4

DCECombPositionType
-------------------

Types of DCE Combined Position Strategies.

**Definition**

**Description**

YD\_CPT\_DCE\_FuturesOffset=0

Futures Locked Strategy

YD\_CPT\_DCE\_OptionsOffset=1

Options Locked Strategy

YD\_CPT\_DCE\_FuturesCalendarSpread=2

Futures Spread Strategy

YD\_CPT\_DCE\_FuturesProductSpread=3

Futures Inter-product Strategy

YD\_CPT\_DCE\_BuyOptionsVerticalSpread=4

Buying options with Vertical Spread Strategy

YD\_CPT\_DCE\_SellOptionsVerticalSpread=5

Selling options with Vertical Spread Strategy

YD\_CPT\_DCE\_OptionsStraddle=7

Options Straddle Strategy

YD\_CPT\_DCE\_OptionsStrangle=8

Options Strangle Strategy

YD\_CPT\_DCE\_BuyOptionsCovered=9

Buying call options while buying corresponding futures contract

YD\_CPT\_DCE\_SellOptionsCovered=10

Buying pull options while selling corresponding futures contract

SSE/SZSECombPositionType
------------------------

Types of SSE/SZSE Combined Position Strategies.

**Definition**

**Description**

YD\_CPT\_StockOption\_CNSJC=100

Call Bull Spread

YD\_CPT\_StockOption\_CXSJC=101

Call Bear Spread

YD\_CPT\_StockOption\_PNSJC=102

Put Bull Spread

YD\_CPT\_StockOption\_PXSJC=103

Put Bear Spread

YD\_CPT\_StockOption\_KS=104

Stock Option Straddle

YD\_CPT\_StockOption\_KKS=105

Stock Option Strangle

ConnectionSelection
-------------------

Methods to select which seat to connect.

**Definition**

**Description**

YD\_CS\_Any=0

Do not select any fixed seat. ydServer submits orders to the exchange as fast as possible, according to available connections

YD\_CS\_Fixed=1

Select a fixed seat to connect. If this seat is not available due to network problems, ydServer will wait until the seat is available

YD\_CS\_Prefered=2

Select a preferred seat to connect. If this seat is unavailable due to network problems, ydServer will connect to any available seat to send trading orders

AlterMoneyType
--------------

For admins only

**Definition**

**Description**

YD\_AM\_ModifyUsage=0

Modify usage of the capital

YD\_AM\_Deposit=1

Deposit money

YD\_AM\_FrozenWithdraw=2

Freeze the capital before withdrawing money

YD\_AM\_CancelFrozenWithdraw=3

Cancel freezing the capital before withdrawing money

YD\_AM\_WithdrawYD\_CS\_Prefered=4

Withdraw money

YD\_AM\_DepositTo=5

Deposit money to designated object

YD\_AM\_WithdrawTo=6

Withdaw moeny to designated object

YD\_AM\_ForceModifyUsage=7

Force modifying usage of the capital

Request Type
------------

For admins only.

**Definition**

**Description**

YD\_RT\_ChangePassword=0

Change password

YD\_RT\_SetTradingRight=1

Change trading permissions

YD\_RT\_AlterMoney=2

Modify the capital and deposit/withdrawal

YD\_RT\_SeletionConnection=3

Methods to select which seat to connect

YD\_RT\_AdminTrading=4

Whether the administrator has the permission to trade

YD\_RT\_UpdateMarginRate=5

Update the margin rate (ETF options only)

API Event
---------

**Definition**

**Description**

YD\_AE\_TCPTradeConnected=0

Event pushed when connected to TCP Trade Server

YD\_AE\_TCPTradeDisconnected=1

Event pushed when disconnected to TCP Trade Server

YD\_AE\_TCPMarketDataConnected=2

Event pushed when connected to TCP Market Data Server

YD\_AE\_TCPMarketDataDisconnected=3

Event pushed when disconnected to TCP Market Data Server

YD\_AE\_ServerRestarted=4

Event pushed when server restarts, before ydClient closes

YD\_AE\_ServerSwitched=5

Event pushed after switching to secondary(standby) server.

Account Flag
------------

Account function indicator.

**Definition**

**Description**

YD\_AF\_SelectConnection=1

Allow to send optimal result of selected connection to ydServer

YD\_AF\_AutoMakeCombPosition=2

Allow ydServer to help make combined positions

YD\_AF\_BareProtocol=4

Allow the user to use raw protocol order

YD\_AF\_DisableSelfTradeCheck=8

Forbid the user to check self trade

Base Price Type
---------------

Determine to use which base price to calculate margin.

**Definition**

**Description**

YD\_CBT\_PreSettlementPrice=0

Previous day's settlement price

YD\_CBT\_OpenPrice=1

Price for eastablishing a position

YD\_CBT\_LastPrice=2

Last price

YD\_CBT\_MarketAveragePrice=3

Market average price

YD\_CBT\_MaxLastPreSettlementPrice=4

Max(previous day's settlement price, last price)

YD\_CBT\_OrderPrice=5

Order's price

YD\_CBT\_None=6

None (Base price ignored)

YD\_CBT\_SamePrice=7

When freezing margin, uses maintenance margin as base price.

IDType in IDFrom Exchange
-------------------------

Types of ID returned by Exchanges.

**Definition**

**Description**

YD\_IDT\_NormalOrderSysID=0

Standard order's system ID

YD\_IDT\_QuoteDerivedOrderSysID=1

System ID of the order derived by a quotation

YD\_IDT\_OptionExecuteOrderSysID=2

Option execution order's system ID

YD\_IDT\_OpitionAbandonExecuteOrderSysID=3

Abandoning option execution order's system ID

YD\_IDT\_RequestForQuoteOrderSysID=4

System ID of the quotation request order

YD\_IDT\_CombPositionOrderSysID=5

Combined position order's system ID

YD\_IDT\_TradeID=128

Trade ID

YD\_IDT\_CombPositionDetailID=129

Combined Position's detail ID

General Risk Param Types
------------------------

Types of general risk parameters.

**Definition**

**Description**

YD\_GRPT\_OptionLongPositionCost=1

Long option position limit

YD\_GRPT\_TradePositionRatio=2

Ratio between volume and open interest

YD\_GRPT\_OrderCancelRatio=3

Ratio between order and cancel order

Data Structure
==============

Data structures begin with "SystemUse" are ydApi's internal fields. Users should not make any assumption about or modify those fields.

User-defined fields will reset to 0 before YDListener::notifyFinishInit() is called. Users can save some business logic values in those fields, and ydApi won't modify them. In C++, data members declared as mutable can be modified even though they are the part of an object declared as const. Simple data can be saved in UserFloat, UserInt or UserInt2; complex data can be saved in class or data structure that its address is saved in \*pUser.

YDSystem Param
--------------

System parameters, YDString is a character array.

**Definition**

**Description**

YDString Name

Parameter name

YDString Target

Parameter target

YDString Value

Parameter value

**Name**

**Target**

**Values**

MarginLowerBoundaryCoef

MarginCalcMethod1

The Minimal Security Coefficient used by Stock Market Index Option Margin equation, float, default value is 0.5

MarginBasePrice

Futures or Options

Base price used when calculating margin of short futures or put options: (default value is 0)  
0: previous day's settlement price  
1: price to establish a position  
2: last price  
3: market average price  
4: Max(last price, previous day's settlement price )  
In CTP system, futures use 1 and options use 4.

OrderMarginBasePrice

Futures or Options

Base price used when calculating frozen margin of establishing a future or selling option position:  
0: previous day's settlement price  
5: order price ( use limit up if it is a market order)  
7: use MarginBasePrice  
In CTP system's rule, both futures and options use 1.  
Note, in option execution order, the frozen margin is always based on previous day's settlement price

SellOrderPremiumBasePrice

Options

Base price used when calculating premium of establishing a selling option position  
0: previous day's settlement price  
5: quotation (use limit down if it is a market order)  
6: no reverse-frozen  
In CTP system, default value is 6.  
Note, in eastablishing a buying option position, frozen premium is always based on order price(use limit up if it is a market order) .

YDExchangeTradeConstrain
------------------------

Trading limitations.

**Definition**

**Description**

int ExchangeOrderSpeed

the amount of orders sent during a period of time

int ExchangeOrderPeriod

a period of time, in seconds

YDExchange
----------

Information of the exchange.

**Definition**

**Description**

YDExchangeID ExchangeID

Exchange's ID, string. For examples, CFFEX, SHFE, DCE and CZCE.

int ExchangeRef

Exchange's reference ID

int ConnectionCount

The amount of ydServer's connected seats

int ProductRefStart

Start number of product's reference ID

int ProductRefEnd

End number of product's reference ID

bool UseTodayPosition

Whether to identify as today's positions( SHFE/INE only)

bool UseArbitragePosition

Whether to identify as arbitrage positions (SHFE/INE only)

bool CloseTodayFirst

Whether to use today's position to liquidate first (CFFEX only)

bool SingleSideMargin

Whether to apply collecting margin on only the side with the larger trading margin requirement��SHFE , INE, CFFEX and DCE��

int OptionExecutionSupport

Whether the exchange supports option exercise or not. 0 indicates unsupported, 1 indicates support without risk management function, 2 indicates support with risk management function.

int OptionAbandonExecutionSupport

Whether the exchange supports abandoning option execution or not. 0 indicates unsupported, -1 indicates support without risk management function, 2 indicates support with risk management function.

int QuoteVolumeRestriction

Whether the exchange has a limit on quotation volume or not. 0 indicates only allowing single-side quotations, 1 indicates allowing two-sided quotations with different amounts, 2 indicates allowing two-sided quotations with the same amount.

short FutureMarginType

Type of future margin. 0 indicates long margin and short margin calculated separately. 1 indicates max( long margin, short margin), the method which CZCE uses.

bool CombineHedgeSpecPosition

"Combining hedge and speculation positions and then handle" indicator, which CZCE uses. When this indicator equals true, the hedge position can be handled as a speculation position.

bool TradeStockOptions

Whether support ETF Options or not

YDExchangeTradeConstrain  
TradeConstrains\[YD\_MaxHedgeFlag\]

Speculator, arbitrageur, hedger and market maker 's trade constraint of Exchange

mutable void \*pUser  
mutable double UserFloat  
mutable int UserInt1  
mutable int UserInt2

User-defined fields.

YDTradeConstrain
----------------

Trade limitation

**Definition**

**Description**

int OpenLimit

Maximum openning of positions, -1 indicates no limitation. Unit: lot

int CancelLimit

Maximum cancelation of orders, -1 indicates no limitation. Unit: number of orders

int PositionLimit

Maximum position held, -1 indicates no limitation. Unit: lot

int TradeVolumeLimit

Maximum trading volume, -1 indicates no limitation. Unit: lot

int DirectionOpenLimit\[2\]

Maximum opening of long and short positions, -1 indicates no limitation. Unit: lot

int DirectionPositionLimit\[2\]

Maximum long and short position held, -1 indicates no limitation. Unit: lot

Exchange trade constraint means that constraint on the summation of every contract of an account in the Exchange. For example, if OpenLimit is 500, the volume of the opening of positions in all orders of that day of that account can not exceed 500 lots.

Explanations of parameters:

*   HedgeFlag: Types of trading strategies. 1 for speculation, 2 for arbitrage and 3 for hedge. If set as 0 or -1, it indicates that this constraint is applicable for every types of trading strategies.
*   ProductOpenLimit: Limitation on total open positions established by a trading account of a product
*   ProductCancelLimit: Limitation on total cancel orders submitted by a trading account of a product.
*   ProductPositionLimit: Limitation on total position held by allowed a trading account of a product.
*   ProductTradeVolumeLimit: Limitation on trading volume of a product.
*   ProductBuyOpenLimit: Limitation on long open position established by a trading account of a product.
*   ProductSellOpenLimit: Limitation on short open position by a trading account of a product.
*   ProductLongPositionLimit: Limitation on long position held by a trading account of a product.
*   ProductShortPositionLimit: Limitation on short position held by a trading account of a product.

We used a "pessimistic" algorithm to control transaction flow. Take ProductCancelLimit as an example, if a user submits a limit order, then total cancel orders will increase by the amount of that limit order immediately; if the limit order is filled or rejected by the exchange, the total cancel orders will decrease by the amount of that limit order. This "pessimistic" algorithm is also adopted by ProductOpenLimit, ProductPositionLimit and ProductTradeVolumeLimit.

YDProduct
---------

Product information.

**Definition**

**Description**

YDProductID ProductID

Product ID

int ProductRef

Product's reference ID

int ExchangeRef

Exchange's reference ID

int ProductClass

Product's class(e.g.,YD\_PC\_Futures,YD\_PC\_Options��YD\_PC\_Combination)

int Multiple

Contract size

double Tick

Tick size

double UnderlyingMultiply

Underlying contract size(options only)

int MaxMarketOrderVolume

Max Market Order Volume

int MinMarketOrderVolume

Min Market Order Volume

int MaxLimitOrderVolume

Max Limit Order Volume

int MinLimitOrderVolume

Min Limit Order Volume

int InstrumentRefStart

Start number of instrument reference id

int InstrumentRefEnd

End number of instrument reference id

int SystemUse7\[32\]

System field

const YDProduct \*m\_pMarginProduct

Calculate product's margin

const YDExchange \*m\_pExchange

Pointer of YDExchange

mutable void \*pUser  
mutable double UserFloat  
mutable int UserInt1  
mutable int UserInt2

User-defined fields. Not available when YDExtendedAPI is in use

YDInstrument
------------

Instrument information.

**Definition**

**Description**

YDInstrumentID InstrumentID

Instrument ID

int InstrumentRef

Instrument's reference ID

int ProductRef

Product's reference ID

int ExchangeRef

Exchange's reference ID

int ProductClass

Product class (e.g., YD\_PC\_Futures, YD\_PC\_Options, YD\_PC\_Combination)

int DeliveryYear

Delivery year

int DeliveryMonth

Delivery month

int MaxMarketOrderVolume;

Max Market Order Volume

int MinMarketOrderVolume

Min Market Order Volume

int MaxLimitOrderVolume

Max Limit Order Volume

int MinLimitOrderVolume

Min Limit Order Volume

double Tick

Tick size

int Multiple

Contract size

int ExpireDate

Expiration Date (Same as the last trading day)

double StrikePrice

Strike price if it is an underlying futures instrument.

int UnderlyingInstrumentRef

Underlying reference ID

int OptionsType

Types of option (e.g., YD\_OT\_NotOption, YD\_OT\_CallOption, YD\_OT\_PutOption)

double UnderlyingMultiply

Underlying mutiplier (Options only)

int SystemUse7\[32\]

System field

bool SingleSideMargin

Whether to apply collecting margin on only the side with the larger trading margin requirement (SHFE , INE, CFFEX and DCE)

int SystemUse6

System field

short ExpireTradingDayCount

Days till last trading day

int MarginCalcMethod

Methods to calculate margin: (Options only)  
1: CFFEX Index Options  
2: SHFE/INE ETF Options  
0: Other Options

YDInstrumentID InstrumentHint

Instrument hint ��SSE/SZSE). e.g.510050C2009M02350

const YDInstrument \*m\_pUnderlyingInstrument;

Pointer to a YDInstrument (Options)

const YDExchange \*m\_pExchange;

Pointer to a YDExchange

const YDProduct \*m\_pProduct;

Pointer to a YDProduct

const YDMarketData \*m\_pMarketData;

Pointer to a YDMarketData

mutable void \*pUser;  
mutable double UserFloat;  
mutable int UserInt1;  
mutable int UserInt2;

User-defined fields. Not available when YDExtendedAPI is in use.

bool AutoSubscribed

Auto subscribed or not

bool UserSubscribed

User subscribed or not

bool IsStaticMargin

Margin is static or not. True If MarginBasePriceType is YD\_CBT\_PresettlementPrice or YD\_CBT\_OpenPrice. False otherwise.

int MarginBasePriceType

Type of margin base price for margin call (Futures, options)

int OrderPremiumBasePriceType\[2\]

Type of premium base price (Options)

int OrderMarginBasePriceType

Type of margin base price for initial margin (Futures, options)

int MarginBasePriceAsUnderlyingType

Type of margin base price for underlying

double MarginBasePrice

Base price used to get margin

const YDMarginRate  
\*m\_pExchangeMarginRate\[YD\_MaxHedgeFlag\]

Exchange's margin rate(MR).

YDMarketData
------------

Market data information.

**Definition**

**Description**

int InstrumentRef

Instrument reference ID

int TradingDay;

Trading day

double PreSettlementPrice;

Previous day's settlement price

double PreClosePrice;

Previous day's closing price

double PreOpenInterest;

Previous day's open interest

double UpperLimitPrice;

Limit up price

double LowerLimitPrice;

Limit down price

double LastPrice;

Last price

double BidPrice;

Bid price. 0 indicates no bid price.

double AskPrice;

Ask price. 0 indicates no ask price.

int BidVolume;

Bid volume. 0 indicates no bid volume.

int AskVolume;

Ask volume. 0 indicates no ask volume.

double Turnover;

Turnover

double OpenInterest;

Open insterest

int Volume;

Volume

int TimeStamp;

Timestamp, integer, indicates the milliseconds passed from the beginning of the trading day till now. Every trading day begins at 5 pm, so 17:00:00 equals 0 in the timestamp. It does not distinguish between Saturday and Monday.

const YDInstrument \*m\_pInstrument

A pointer of YDInstrument

mutable void \*pUser;  
mutable double UserFloat;  
mutable int UserInt1;  
mutable int UserInt2;

User-defined fields. Not available when YDExtendedAPI is in use.

The timestamp indicates the time from the beginning of the trading day up to this time. Every trading day begins at 5 pm, so 17:00:00 equals 0 in the timestamp. This timestamp is designed base on China's overnight trading. Trading on Friday night belongs to the following Monday. There are three parts of Monday's trading hours: Friday night (0-25199999), Saturday early morning (25200000-34199999) and Monday daytime (34200000-86399999).

YDCombPositionDef
-----------------

Definition of combined position.

**Definition**

**Description**

YDInstrumentID CombPositionID

Combined position ID

int CombPositionRef

Combined position reference ID

int ExchangeRef

Exchange's reference ID

int Priority

Priority

short CombHedgeFlag

Type of combined trading stratgies

short CombPositionType

Type of combined position

const YDExchange \*m\_pExchange

A pointer of YDExchange

const YDInstrument \*m\_pInstrument\[2\]

2-legs position

int PositionDirection\[2\]

Trading direction of 2-legs position

int HedgeFlag\[2\]

Trading type of 2-legs position

int PositionDate\[2\]

Today's position or historical position. Set as historical position by default.

mutable void \*pUser  
mutable double UserFloat  
mutable int UserInt1  
mutable int UserInt2

User-defined fields. Not available when YDExtendedAPI is in use.

YDAccount
---------

Client/Account capital information.

**Definition**

**Description**

int SystemUse1

System field

int AccountRef

Account reference ID

YDAccountID AccountID

Account ID

double PreBalance

Previous day's balance

double WarningLevel1

Warning level used by the monitor; show orange when exceeded

double WarningLevel2

Warning level used by the monitor; show red when exceeded

double MaxMoneyUsage

Maximum usage of the capital

double Deposit

Deposit (Today)

double Withdraw

Withdraw(Today)

double FrozenWithdraw

Frozen withdrawal. Futures company can freeze withdrawal at first preventing users from withdrawing. And then exercise withdrawing and unfreeze withdrawal at last.

int TradingRight

Trading permission

int MaxOrderCount

Maximum order count

int MaxLoginCount

Maximum login count

int LoginCount

login count

int AccountFlag

Account flag

int SystemUse3

System field

mutable void \*pUser  
mutable double UserFloat  
mutable int UserInt1  
mutable int UserInt2

Note: Users need to calculate profit or loss of liquidating position, cash into or out of option instruments, commission, profit or loss of the position, margin used, cash balance and available balance by themselves according to orders, matched trades and current price.

Note: Users need to calculate profit or loss of liquidating position, cash into or out of option instruments, commission, profit or loss of the position, margin used, cash balance and available balance by themselves according to orders, matched trades and current price.

YDPrePosition
-------------

Previous day's position of client.

**Definition**

**Description**

int SystemUse1

System field

int AccountRef

Account reference ID

int InstrumentRef

Instrument reference ID

int PositionDirection

Directions of position. Check datatype "PositionDirection" for more details.

int HedgeFlag

Types of trading stratrgies. See HedgeFlag.

int PrePosition

Previous day's positions

double PreSettlementPrice

Previous day's settlement price

const YDInstrument \*m\_pInstrument

Pointer to a YDInstrument

const YDAccount \*m\_pAccount

Pointer to a YDAccount

YDCombPosition
--------------

Combined position.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3

System fields

int AccountRef

Account reference ID

int CombPositionRef

Combined position reference ID

int Position

Position

in CombPositionDetailID

Combined position detail ID

int SystemUse6

System field

YDInputOrder
------------

Order.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3  
int SystemUse4  
int SystemUse5

System fields

char Direction

Trading directions: YD\_D\_Buy, YD\_D\_Sell

char OffsetFlag

Offset indicator

char HedgeFlag

Type of trading stratrgies

char ConnectionSelectionType;

Type of selection methods for connection

union { double Price; struct { int CombPositionDetailID; int SystemUse6;};}

Price of limit order; or combined position's detail ID when spliting in SSE/SZSE

int OrderVolume

Volume of orders

int OrderRef

Order's local reference ID. For orders are not generated by ydServer, their local reference IDs are set as -1.

char OrderType

Types of order: Limit/Fill And Kill(FAK)/ Market(MKT)

char YDOrderFlag;

Type of YDOrder

char ConnectionID;

Connection ID

char RealConnectionID;

Actual connection ID when using YD\_CS\_Any or YD\_CS\_Perfered to submit orders. This field is filled by ydServer. You can set it as 0.

int ErrorNo;

Return the error code if order is rejected. Otherwise, return 0.

InputOrder is used for standard order, option execution order, abandoning option execution order, quotation and making/splitting combined position.  
Instruction for filling those fields is listed below:  
Note that except for the trading direction of option execution order and abandoning option execution order, every field that is not required must be filled with 0. You don't need to consider this situation if you always cleared inputOrder to 0 at the beginning.

**Order Types**

**Standard Order**

**Option Execution Order**

**Abandoning Option Execution Order**

**Quotation**

**Making/Splitting Combined Poistion (DCE, SSE, SZSE)**

**YDOrderFlag**

YD\_YOF\_Normal

YD\_YOF\_OptionExecute

YD\_YOF\_OptionAbandonExecute

YD\_YOF\_RequestForQuote

YD\_YOF\_CombPosition

**pInstrument**

Must be assigned

Must be assigned

Must be assigned

Must be assigned

Must be assigned

**pAccount**

Must be assigned (Null indicates user him or her self)

Must be assigned (Null indicates user him or her self)

Must be assigned (Null indicates user him or her self)

Must be assigned (Null indicates user him or her self)

Must be assigned (Null indicates user him or her self)

**OrderType**

Must be assigned

YD\_ODT\_Limit

YD\_ODT\_Limit

YD\_ODT\_Limit

Set as 0

**Direction**

Must be assigned

YD\_D\_Sell

YD\_D\_Sell

YD\_D\_Buy

Must be assigned (making or spliting)

**OffsetFlag**

Must be assigned

Must be assigned

Must be assigned

YD\_OF\_Open

Set as 0

**HedgeFlag**

Must be assigned

Must be assigned

Must be assigned

Must be assigned

Must be assigned

**Price/CombPositionDetailID**

Must be assigned if order type is not market order

Set as 0

Set as 0

Set as 0

Must be assigned if spliting in SSE/SZSE, 0 otherwise.

**OrderVolume**

Must be assigned

Must be assigned

Must be assigned

Set as 0

Must be assigned

**OrderRef**

Any

Any

Any

Any

Any

**ConnectionSelectionType**

Must be assigned

Must be assigned

Must be assigned

Must be assigned

Must be assigned

**ConnectionID**

Need to be assigned if connection type is notYD\_CS\_Any

Need to be assigned if connection type is notYD\_CS\_Any

Need to be assigned if connection type is notYD\_CS\_Any

Need to be assigned if connection type is notYD\_CS\_Any

Need to be assigned if connection type is notYD\_CS\_Any

**RealConnectionID**

Set as 0

Set as 0

Set as 0

Set as 0

Set as 0

**ErrorNo**

Set as 0

Set as 0

Set as 0

Set as 0

Set as 0

Different exchanges have different rules about option execution or abandoning option execution.

**Exchanges**

**Option Execution**

**Abandoning Option Execution**

SHFE

Available, and always auto-offset the option execution

Available

INE

Appears on the interface, but there is no such function currently.

Appears on the interface, but there is no such function currently.

CFFEX

Not available

Not available

DCE

Available, we only called DCE's ReqTraderOptExec, auto-offset or not is decided by DCE.

Not available. Althought it is on DCE's interface, its function is not same.

YDOrder
-------

Order return.

**Definition**

**Description**

YDInputOrder

Superclass of YDOrder. Input order

int ExchangeRef

Exchange reference ID

int OrderSysID

Order's system ID of Exchanges

int OrderStatus

Order's status. Check OrderStatus for more details.

int TradeVolume

Filled order volume

int InsertTime

Insert-time of order, an integer indicates the milliseconds passed from the beginning of the trading day to now. Every trading day begins at 5 pm, so 17:00:00 equals 0 in the timestamp. It does not distinguish between Saturday and Monday.

int OrderLocalID

Order Local ID, in asceding order, starting number is based on the order's maximum local reference ID which returned by.

YDCancelOrder
-------------

Cancel order.

**Definition**

**Description**

int SystemUse1;  
int SystemUse2;  
int SystemUse3;  
int SystemUse4;

System fields

int OrderSysID;

Order's system ID of Exchanges

char SystemUse5

System fields

char ConnectionSelectionType

Type of selection methods for connection

char ConnectionID

Connection ID

char YDOrderFlag

Orders' indicator.

Cancellation requests are used for standard cancel order, option execution cancel order, and option abandoned execution cancel order. The required values of each field are listed below:

**Order Type**

**Standard Order**

**Option Execution Order**

**Abandoning Option Execution Order**

**YDOrderFlag**

YD\_YOF\_Normal

YD\_YOF\_OptionExecute

YD\_YOF\_OptionAbandonExecute

**pExchange**

Must be assigned

Must be assigned

Must be assigned

**pAccount**

Must be assigned (Null indicates user him or her self)

Must be assigned (Null indicates user him or her self)

Must be assigned (Null indicates user him or her self)

**OrderSysID**

Must be assigned

Must be assigned

Must be assigned

**ConnectionSelectionType**

Must be assigned

Must be assigned

Must be assigned

**ConnectionID**

Must be assigned if not YD\_CS\_Any

Must be assigned if not YD\_CS\_Any

Must be assigned if not YD\_CS\_Any

YDFailedCancelOrder
-------------------

Failed cancel order.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3

System fields

int AccountRef

Account reference ID

YDSysOrderID OrderSysID

Order's system ID of Exchanges

char ExchangeRef

Exchange reference ID

char SystemUse6  
char SystemUse7

System fields

char YDOrderFlag

Orders' indicator

int ErrorNo

Error code

char SystemUse8

System field

YDTrade
-------

Matched/Affermitive trade return.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3

System fields

int AccountRef

Account reference ID

int InstrumentRef

Instrument reference ID

char Direction

Trading directions��YD\_D\_Buy, YD\_D\_Sell

char OffsetFlag

Offset indicator. For more information, see OffsetFlag.

char HedgeFlag

Trading types indicator. For more information, see HedgeFlag.

char SystemUse6

System field

int TradeID

Trade ID

int OrderSysID

Order's system ID of Exchanges

double Price

Price

int Volume

Volume

int TradeTime

Trading time of an order, an integer, which indicates the milliseconds passed from beginning of trading day till now. Every trading day begins at 5pm, so 17:00:00 equals 0 in timestamp. It does not distinguish between Saturday and Monday.

double Commission

Commission rate

int OrderLocalID

Order's local ID, in asceding order, starting number is based on the order's maximum local reference ID which returned by all seat connections.

int OrderRef

Order's local reference ID.

YDInputQuote
------------

Price quotation.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3  
int SystemUse4  
int SystemUse5

System fields

char BidOffsetFlag

Bid offset flag. Must be assigned.

char BidHedgeFlag

Bid hedge flag. Must be assigned.

char AskOffsetFlag

Ask offset flag. Must be assigned.

char AskHedgeFlag

Ask hedge flag. Must be assigned.

double BidPrice

Bid price. Must be assigned.

double AskPrice

Ask price. Must be assigned.

int BidVolume

Bid volume. Must be assigned, 0 indicates no need to bid.

int AskVolume

Ask volume. Must be assigned, 0 indicates no need to ask.

int OrderRef

Order reference ID. Must be assigned during quotation.

char ConnectionSelectionType

Type of selection methods for connection. See ConnectionSelection.

char ConnectionID

Connection ID

char RealConnectionID

Actual Connection ID. Set as 0 when submitting the quotation

char YDQuoteFlag

Quotation indicator

int Reserved

Reserved system field

int ErrorNo

Error code. Show error code if order is denied, 0 otherwise. Fill in 0 when submitting an order.

Different exchanges have different rules about BidVolume and AskVolume:

*   For SHFE and INE, BidVolume and AskVolume must be equal to each other, and are greater than 0.
*   For CFFEX, BidVolume and AskVolume must be greater than 0.
*   For DCE, at least one of BidVolume and AskVolume must be greater than 0, and the other is greater than or equal to 0.

Different exchanges have different rules about BidHedgeFlag and AskHedgeFlag:

*   For CFFEX, exchange interface acutally only supports when BidHedgeFlag and AskHedgeFlag are same. When submitting orders to the exchange, client number based on trading types is used.
*   For SHFE, INE and DEC, BidHedgeFlag and AskHedgeFlag can be different.

YDQuote
-------

Quotation return.

**Definition**

**Description**

YDInputQuote

Quotation from the market maker

int ExchangeRef

Exchange's reference ID

int QuoteSysID

Quote's system ID of Exchanges

int BidOrderSysID

Bid order's system ID of Exchanges

int AskOrderSysID

Ask order's system ID of Exchanges

YDCancelQuote
-------------

Cancel quotation.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3  
int SystemUse4

System fields

int QuoteSysID

Quote's system ID of Exchanges. Must be assigned.

char SystemUse5

System field

char ConnectionSelectionType

Type of selection methods for connection

char ConnectionID

Connection ID. Must be specified if not YD\_CS\_Any

char SystemUse6

System field

YDFailedCancelQuote
-------------------

Failed cancel quotation.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3

System fields

int AccountRef

Account reference ID

YDSysOrderID QuoteSysID

Quote's system ID of Exchanges

char ExchangeRef

Exchange's reference ID

char SystemUse6  
char SystemUse7  
char SystemUse8

System fields

int ErrorNo

Error code

int SystemUse9

System field

YDRequestForQuote
-----------------

Request for quotation.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3  
int SystemUse4

System fields

int InstrumentRef

Instrument reference ID

int RequestTime

Request time of quotation

int RequestForQuoteID

Quotation ID (DCE only)

char SystemUse6;

System field

YDMarginRate
------------

Margin rate.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3  
int SystemUse4

System fields

int HedgeFlag

Types of trading stratgies. See HedgeFlag

int SystemUse5

System field

union {double LongMarginRatioByMoney; double PutMarginRatioByMoney;}

Calculate margin rate of long futures or put options by money

union {double LongMarginRatioByVolume; double PutMarginRatioByVolume;}

Calculate margin rate of long futures or put options by volume

union {double ShortMarginRatioByMoney; double CallMarginRatioByMoney;}

Calculate margin rate of short futures or call options by money

union {double ShortMarginRatioByVolume; double CallMarginRatioByVolume;}

Calculate margin rate of short futures or call options by money

const YDInstrument \*m\_pInstrument

Pointer of YDInstrument

const YDProduct \*m\_pProduct

Pointer of YDProduct

const YDAccount \*m\_pAccount

Pointer of YDAccount

YDCommissionRate
----------------

Commission rate.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3  
int SystemUse4

System fields

int HedgeFlag

Types of trading stratgies. See HedgeFlag

int SystemUse5

System field.

double OpenRatioByMoney

Calculate commission rate of openning position by money.

double OpenRatioByVolume

Calculate commission rate of opening position by volume.

double CloseRatioByMoney

Calculate commission rate of liquidating previous day's position by money.

double CloseRatioByVolume

Calculate commission rate of liquidating previous day's position by volume.

double CloseTodayRatioByMoney

Calculate commission rate of liquidating today's position by money.

double CloseTodayRatioByVolume

Calculate commission rate of liquidating today's position by volume.

const YDInstrument \*m\_pInstrument

Pointer of YDInstrument

const YDProduct \*m\_pProduct

Pointer of YDProduct

const YDAccount \*m\_pAccount;

Pointer of YDAccount

YDIDFromExchange
----------------

Exchange ID.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3

System fields

int AccountRef

Account reference ID

int ExchangeRef

Exchange reference ID

int IDType

ID Type. For more information, see IDType.

int IDInSystem

System internal ID

int SystemUse6

System field

char IDFromExchange\[24\]

Exchange ID

YDUpdateMarginRate
------------------

Update margin rate.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
int SystemUse3

System fields

int AccountRef

Account's reference ID

int ProductRef

Product's reference ID

int InstrumentRef

Instrument's reference ID

int UnderlyingInstrumentRef

Underlying Instrument's reference ID

int HedgeFlagSet

Set of HedgeFlag

int ExpireDate

Expiration date

int Multiple

Contract size

int OptionTypeSet

Set of Option types

int SystemUse4

System field

union {double LongMarginRatioByMoney; double PutMarginRatioByMoney;}

Calculate margin rate of long futures or put options by money

union {double LongMarginRatioByVolume; double PutMarginRatioByVolume; double BaseMarginRate;}

Calculate margin rate of long futures or put options by volume, or calculate by base margin rate.

union {double ShortMarginRatioByMoney; double CallMarginRatioByMoney; double LinearFactor;}

Calculate margin rate of short futures or call options by money, or calculate by linear factor.

union {double ShortMarginRatioByVolume; double CallMarginRatioByVolume; double LowerBoundaryCoef;}

Calculate margin rate of short futures or call options by volume, or calculate by volume.

YDAccountExchangeInfo
---------------------

Account's exchange information and trading permission.

**Definition**

**Description**

int SystemUse1

System field

int AccountRef

Account reference ID

int ExchangeRef

Exchange reference ID

int TradingRight;

Trading permission of a specific exchange. See TradeRight

const YDAccount \*m\_pAccount;

Pointer of YDAccount

const YDExchange \*m\_pExchange;

Pointer of YDExchange

mutable void \*pUser;  
mutable double UserFloat;  
mutable int UserInt1;  
mutable int UserInt2;

User-defined fields. Not available when YDExtendedAPI is in use.

YDAccountProductInfo
--------------------

Account's product information.

**Definition**

**Description**

int SystemUse1

System field

int AccountRef

Account reference ID

int ProductRef

Product reference ID

int TradingRight;

Trading right of a specific product. See TradeRight

YDTradeConstraint TradingConstraints\[YD\_MaxHedgeFlag\]

Product Trading Right. See YDTradeConstraint.

const YDAccount \*m\_pAccount;

Pointer of YDAccount \*m\_pAccount

const YDProduct \*m\_pProduct;

Pointer of YDProduct \*m\_pProduct

mutable void \*pUser  
mutable double UserFloat  
mutable int UserInt1  
mutable int UserInt2

User-defined fields. Not available when YDExtendedAPI is in use.

YDAccountInstrumentInfo
-----------------------

Account's instrument information.

**Definition**

**Description**

int SystemUse1

System field

int AccountRef

Account reference ID

int InstrumentRef

Instrument reference ID

int TradingRight

Trading right of a specfic instrument. See TradeRight

YDTradeConstraint TradingConstraints\[YD\_MaxHedgeFlag\]

Instrument Trading Right. See YDTradeConstraint

const YDAccount \*m\_pAccount

YDAccount

const YDInstrument \*m\_pInstrument

YDProduct

const YDMarginRate  
\*m\_pMarginRate\[YD\_MaxHedgeFlag\];

Pointer of MaxHedgeFlag's margin rate

const YDCommissionRate  
\*m\_pCommissionRate\[YD\_MaxHedgeFlag\];

Pointer of MaxHedgeFlag's commission rate

mutable void \*pUser;  
mutable double UserFloat;  
mutable int UserInt1;  
mutable int UserInt2;

User-defined fields. Not avaialble when YDExtendedAPI.

YDGeneralRiskParam
------------------

General risk parameters.

**Definition**

**Description**

int SystemUse1  
int SystemUse2  
intSystemUse<3>

Parameter name

int AccountRef

Account reference ID

int GeneralRiskParamType

Type of general risk parameters

int ExtendedRef

Extended reference ID

double FloatValue

Floating value

int IntValue1

Integer value 1

int IntValue2

Integer value 2

YDExtendedOrder
---------------

Extended information of orders. YDExtendedOrder is a subclass of YDOrder. Extended fields are listed below:

**Definition**

**Description**

union {const YDInstrument \*m\_pInstrument; const YDCombPositionDef \*m\_pCombPositionDef;};

Pointer points to YDInstrument or YDCombPositionDef.

const YDAccount \*m\_pAccount;

Pointer points to YDAccount

const YDInstrument \*m\_pInstrument2

Pointer points to YDInstrument

YDExtendedTrade
---------------

Extended information of matched trade return. YDExtendedTrade is a subclass of YDTrade. Extended fields are listed below:

**Definition**

**Description**

const YDInstrument \*m\_pInstrument

Pointer of YDInstrument. See YDInstrument.

const YDAccount \*m\_pAccount;

Pointer of YDAccount. See YDAccount.

YDExtendedQuote
---------------

Extended information of quotation. ExtendedQuote is a subclass of YDQuote. Extended fields are listed below:

**Definition**

**Description**

bool BidOrderFinished

Whether bid order is finished or not

bool AskOrderFinished

Whether ask order is finished or not

const YDInstrument \*m\_pInstrument

Pointer of YDInstrument. See YDInstrument.

const YDAccount \*m\_pAccount;

Pointer of YDAccount. See YDAccount.

YDExtendedRequestForQuote
-------------------------

Extended information of request for quotation. YDExtendedRequestForQuote is a subclass of YDRequestForQuote. Extended field is listed below:

**Definition**

**Description**

const YDInstrument \*m\_pInstrument

Pointer of YDInstrument. See YDInstrument.

YDExtendedAccount
-----------------

Extended information of an account.

**Definition**

**Description**

double CloseProfit

Profit or loss of liquidating position

double CashIn

Premium

double OtherCloseProfit

Profit or loss of liquidating position (Other than future instrument)

double Commission

Commission

double Margin

Margin

double PositionProfit

Porfit and loss of the position

double Balance

Cash balance

double Available

Available balance (Not counted as loss of the position)

unsigned UsedOrderCount

User's order count

int Useless

System fields

const YDAccount \*m\_pAccount

Pointer of YDAccount

double ExecMargin

Option exercise margin

double OptionLongPositionCost

Cost of option long positon

double OptionLongPositionCostLimit

Limit of option long position

double useable(void) const

Available balance (Counted as loss of the position)

bool canUse(double value) const

Determine if balance is enough

YDExtendedPosition
------------------

Extended information of the position.

**Definition**

**Description**

int PositionDate

Today's or previous day's position (SHFE & INE)

int PositionDirection

Trading diretions of positions

int HedgeFlag

Types of trading stratgries

int OpenFrozen

Open Frozen

int Position

Position

int PositionByOrder

Volume of positions base on order

int YDPositionForCommission

Position used for calculating commission.  
YD PositionForCommission <= Position is always true.  
For SHFE and INE:  
If PositionDate is YD\_PSD\_Today, YDPositionForCommission is always 0.  
If PositionDate is YD\_PSD\_History, YDPositionForCommission is always 0.  
For other exchanges, YDPositionForCommssion might be greater than 0 and less than Poistion.

int CloseFrozen

Close frozen

int PossibleOpenVolume

Possible total volume of open position

int ExecFrozen

Exercise frozen

double TotalOpenPrice

Total open price

double Margin

Margin

double PositionProfit

Profit or loss of the position

double MarginPerLot

Margin per lot (calculated base on previous day's settlement price)

double CloseProfit

Profit or loss of liquidating position

double OtherCloseProfit

Profit or loss of liquidating position

const YDAccountInstrumentInfo \*m\_pAccountInstrumentInfo

Pointer of YDAccountInstrumentInfo

CPositionDetail \*PositionDetailList

Linked list used to calculated profit or loss of the position.

void \*Padding1

Placeholder

int TotalCombPositions

Total combined positions

int Padding2

Placeholder

unsigned CombPositionCount

number of combined position records

const YDAccount \*getAccount(void) const

return current position's account

const YDInstrument \*getInstrument(void) const

return current position's instrument

const YDMarginRate \*getMarginRate(void) const

return margin rate of current position

const YDCommissionRate \*getCommissionRate(void) const

return commission rate of current position

double OpenPrice(void) const

return average price of openning position of current position. This method won't be available in later versions, please use getOpenPrice instead.

double getOpenPrice(void) const

return average price for establishing position of current positions.

int getYDPosition() const

return previous day's position

### CPositionDetail

**Definition**

**Description**

CpositionDetail \*m\_pNext

Pointer points to next position's detail in linked list.

double Price

Price for establishing a position

int Volume

Volume of position

int TradeID

Trade ID. Trade ID smaller than 0 indicates previous day's position. Previous day's position only appear at front of linked list.

CCZCECombPosition\* m\_pCZCECombPosition

CZCE's combined position

YDExtendedCombPositionDetail
----------------------------

Extended detail of the combined position.

**Definition**

**Description**

const YDAccount \*m\_pAccount

Account

const YDCombPositionDef \*m\_pCombPositionDef

Price for establishing a position

int Position

Position

int CombPositionDetailID

Combined position's detail ID

YDExtendedPositionFilter
------------------------

Information filter of the extended position.

**Definition**

**Description**

int PositionDate

Date of position, -1 indicates all

int PositionDirection

Trading direction of position, -1 indicates all

int HedgeFlag

Types of trading strategies, -1 indicates all

const YDInstrument \*pInstrument

Pointer points to structure YDInstrument, null pointer indicates all.

const YDProduct \*pProduct

Pointer points to structure YDProduct, null pointer indicates all.

const YDExchange \*pExchange

Pointer points to structure YDproduct, null pointer indicates all.

const YDAccount \*pAccount

Pointer pointes to structure YDaccount, null pointer indicates all.

YDOrderFilter
-------------

Information filter of orders.

**Definition**

**Description**

int StartTime

Start time, -1 indicates all. This parameter can be generated by function string2TimeID().

int EndTime

End time, -1 indicates all. This parameter can be generated by function string2TimeID().

int YDOrderFlags

YDOrderFlag can be set by operation bitwise AND. If need to search YD\_YOF\_QuoteDerived, then YDOrderFlags=1<

const YDCombPositionDef \*pCombPositionDef

Pointer points to structure YDCombPositionDef, null pointer indicates all.

const YDInstrument \*pInstrument

Pointer points to structure YDInstrument, null pointer indicates all.

const YDProduct \*pProduct

Pointer points to structure YDProduct, null pointer indicates all.

const YDExchange \*pExchange

Pointer points to structure YDExchange, null pointer indicates all.

const YDAccount \*pAccount

Pointer points to structure YDAccount, null pointer indicates all.

YDQuoteFilter
-------------

Information filter of quotations.

**Definition**

**Description**

const YDInstrument \*pInstrument

Pointer points to structure YDInstrument, null pointer indicates all.

const YDProduct \*pProduct

Pointer points to structure YDProduct, null pointer indicates all.

const YDExchange \*pExchange

Pointer points to structure YDExchange, null pointer indiciates all.

const YDAccount \*pAccount

Pointer points to structure YDAccount, null pointer indicates all.

YDTradeFilter
-------------

Information filter of matched trades.

**Definition**

**Description**

int StartTime

Start time, -1 indicates all. This parameter can be generated by string2TimeID().

int EndTime

End time, -1 indicates all. This parameter can be generated by string2TimeID().

const YDInstrument \*pInstrument

Pointer points to structure YDInstrument, null pointer indicates all.

const YDProduct \*pProduct

Pointer points to structure YDProduct, null pointer indicates all.

const YDExchange \*pExchange

Pointer points to structure YDExchange, null pointer indicates all.

const YDAccount \*pAccount

Pointer points to structure YDAccount, null pointer indicates all.

YDCombPositionDetailFilter
--------------------------

Information filter of the combined position's detail.

**Definition**

**Description>**

const YDCombPositionDef \*pCombPositionDef

Definition of combined position, null pointer indicates all.

const YDAccount \*pAccount

Pointer points to structure YDAccount, null pointer indicates all

const YDInstrument \*pInstrument

Pointer points to structure YDInstrument, null pointer indicates all; if not NULL, the combined position must have at least one leg.

int PositionDirection

Trading direction of positon, 0 indicates both directions; if direction is determined, the combined position must have at least one leg.

bool IncludeSplit

If includes the splitted combined position or not.

API
===

ydApi mainly consists of two parts: extended YDListener's callback functions that receive trading data pushed by ydServer; and ydApi initialization.

ydApi initialization is made of three steps. The first step is to initialize a YDAPI instance; the second step is to call new to create an extended YDListener instance; the third step is to call YDAPI::start to connect the YDApi instance with YDListener instance to launch internal management thread and network communication thread.

Choice of API
-------------

YD offers two types of API: ydApi and ydExtendedApi. In that, ydApi has extremely outstanding performance but needs users to maintain balance and position by themselves; ydExtendedApi can help users maintain balance and position with a cost of performance. Users can choose different API base on their demand.

While using ydExtendedApi, users can get balance by getExtendedAccount, and get position by getExtendedPosition and findExtendedPostion. Order still uses insertOrder, which is inherited from ydApi to achieve high-speed order submission.

About muti-connections
----------------------

There is no particular optimisation for multiple connections during the design and development stages of the YD system. However, many users create multiple ydApi instances to achieve multiple accounts trading at the same time. To avoid loss due to misuse, please be aware of the problems listed below while using a single process to achieve multiple ydApi instances:

*   Don't mingle pointers of different API's instances. Don't use one API instance's pointer in another API's instance. It might cause an unexpected error and affect the normal transaction.
*   YD does not provide the connection ID. Users can distinguish connections by OrderRef. Users can use OrderRef's last 3 bits to present eight different connections.

Global Function
---------------

### makeYDApi

    YDApi *makeYDApi(const char *configFilename);

configFilename is the file name of network connection and CPU/Core related configuration.

This function asks the operating system for space to initialize a YDApi instance.

### makeYDExtendedApi

    YDExtendedApi *makeYDExtendedApi(const char *configFilename);

configFilename is the file name of network connection and CPU/Core related configuration.

This function asks the operating system for space to initialize a YDExtendedApi instance.

### getYDVersion

    const char *getYDVersion(void);

Returns YDAPI's four-part version number. ( i.e. 1.2.0.1)

### getYDNanoTimestamp

    unsigned long long getYDNanoTimestamp();

Returns timestamp, in nanoseconds, by an efficient and accurate method. It can be used to measure the API's running time of submitting an order.

YDListener class:
-----------------

YDListener is a group of threads of network-listener execution. According to different configuration files, the number of threads can be different; however, one thread to receive returns of TCP trading from ydServer is necessary. If need to receive market data from ydServer, one or two more threads is required. Those threads maintain the local database after downstream data are received. And those threads call notify-function to inform ydClient.

### notifyBeforeApiDestory

    virtual void notifyBeforeApiDestory(void);

A callback function called after startDestory is called and before data cleanup.

### notifyAfterApiDestory

    virtual void notifyAfterApiDestory(void);

A callback function called after startDestory is called and after data cleanup.

### notifyEvent

    virtual void notifyEvent(int apiEvent);

When connecting or disconnecting to TCP trading server or TCP market data server, or finding out ydServer restarted, this callback function will notify users.

When ydListener receives the message YD\_AE\_ServerRestarted, ydApi will quit if users do not react to the message. If you do not wish ydApi to stop, you can call startDestory to destroy the ydApi object on your own.

### notifyReadyForLogin

    virtual void notifyReadyForLogin(bool hasLoginFailed);

A callback function called by ydApi after ydClient successfully connected to the ydServer. It means ydClient has already connected to the ydServer's TCP server and is ready for login.

This function has a bool called hasLoginFailed, which tells whether ydClient failed to log in last time. If ydClient tried to log in and failed, ydClient will wait for 3 seconds and then call notifyReadyForLogin. Besides that, hasLoginFailed will be TRUE. Note: The same instance of YDApi class will not support switching usernames during runtime.

### notifyLogin

    virtual void notifyLogin(int errorNo, int maxOrderRef, bool isMonitor);

After the login request is sent, ydServer will check if the username and password are correct, send the login result back to ydClient. After that, ydApi will call function notifyLogin. errorNo indicates whether login succeeds. If success, isMonitor shows if the user is an administrator or not; If true, maxOrderRef will be 0.

maxOrderRef is ydClient's maximum local order reference ID used in one trading day. ydServer allows users to repeat using one username/password many times. However, ydClient needs some mechanism to ensure that there is no duplication in local order reference IDs, like monotonically increasing sequence.

### notifyFinishInit

    virtual void notifyFinishInit(void);

ydApi will call notifyFinishInit only once during runtime after ydClient receives data initialization sent by ydServer. If no order or matched trade return received (like the first run during overnight trading), ydClient can start inital logic here. If YDApi::hasFinishedInit is called after, it will return True; if before, it will return False. The initialized database can be used safely, and user-defined fields will be reset as 0.

Till now, all data in the local initialized database are ready. Required memory is applied. ydClient can use the getX() method of YDApi to retrieve data. The address of the database does not change; thus, ydClient can use pointers without concerns. However, ydClient still needs to calculate margin, commission, balance and position according to order returns and matched trade returns. The data saved in the local database is initialized data, and deposit/withdrawal and trading permissions which the administrator modifies after trading.

ydClient will call notifyFinishInit only once during runtime; moreover, there won't be any callback even if network disconnected. Since ydServer has not been restarted and initialized static data and order/matched trade returns has not changed after disconnection, there is no need to re-initialize after being reconnected to the ydServer. Thereofre, this method is suitable for initializing users' business data.

### notifyCaughtUp

    virtual void notifyCaughtUp(void);

A callback function will notify users that ydClient has already received all orders and matched trade returns. This function must be called after notifyFinishInit. Every time ydClient disconnects with ydServer, notifyCaughtUp will be called after reconnection.

### notifyTradingSegment

    virtual void notifyTradingSegment(const YDExchange *pExchange, int segmentTime);

A callback function will notify users the change of trading segment. To reduce cost of communication, ydServer only notify ydClient the time of change, segmentTime indicates seconds passed from beginning of trading day till now. Every trading day begins at 5pm, so 17:00:00 equals 0 in timestamp. It does not distinguish between Saturday and Monday because of overnight trading.

**Exchanges**

**Day Trading Session**

**Night Trading Session**

CFFEX

09:25:00  
09:29:00  
09:30:00  
11:30:00  
13:00:00  
14:57:00  
15:00:00  
15:15:00

INE

09:00:00  
10:15:00  
10:30:00  
11:30:00  
13:30:00  
15:00:00

20:55:00  
20:59:00  
21:00:00  
23:00:00  
01:00:00  
02:30:00

SHFE

08:55:00  
08:59:00  
09:00:00  
10:15:00  
10:30:00  
11:30:00  
13:30:00  
15:00:00

20:55:00  
20:59:00  
21:00:00  
23:00:00  
01:00:00  
02:30:00

DCE

08:54:50  
08:55:00  
08:58:50  
08:59:00  
08:59:50  
09:00:00  
10:14:50  
10:15:00  
10:29:50  
10:30:00  
11:29:50  
11:30:00  
13:29:50  
13:30:00  
14:59:50  
15:00:00

20:54:50  
20:55:00  
20:58:50  
20:59:00  
20:59:50  
21:00:00  
22:59:50  
23:00:00

### notifyCombPosition

    virtual void notifyCombPosition(const YDCombPosition *pCombPosition,const YDCombPositionDef *pCombPositionDef,const YDAccount *pAccount)

A callback function, called between notifyFinishInit and notifyCaughtUp, recives current combined positions' data to startup.

### notifyOrder & notifyFailedOrder

    
    virtual void notifyOrder(const YDOrder *pOrder, const YDInstrument *pInstrument, const YDAccount *pAccount);
    virtual void notifyFailedOrder(const YDInputOrder *pFailedOrder, const YDInstrument *pInstrument, const YDAccount *pAccount);

Get the order return. For YDApi::insertOrder and YDApi::cancelOrder, both use callback functions notifyOrder and notifyFailedOrder to retrieve the result. The order rejected by ydServer is retrieved by notifyFailedOrder. Order accepted or rejected by exchanges is retrieved by notifyOrder.

For every connected seat, ydServer uses continual local order reference ID. For instance, the system's local order reference ID is 100. When receiving an order, after passing the test, its local order reference ID is set as 101 and is sent to the exchange via seat No.0. After that, another order was received, and its local order reference ID will be placed as 102 even though it was sent to the exchange via seat No.1.

ydServer get order returns and matched trade returns through the exchange's private flow; therefore, it might get order returns and matched trade returns from other systems. To distinguish the difference between returns of YD and returns of other systems, the local order reference ID of returns from other systems are set as -1. If ydServer restarts due to malfunction, the local order reference ID generated by ydServer during the last runtime will be set as -1.

If the exchange rejects an order, pOrder->ErrorNo contains the error code from the exchange. If the exchange accepts it, its ErrorNo is 0.

Order return progressions are different in different exchanges. For limit orders, every exchange would have a return of YD\_OS\_Queuing and have a corresponding return against matched trade or cancellation. For FAK/FOK/MKT order, DCE will still have a return of YD\_OS\_Queuing and have a return of YD\_OS\_Canceled or YD\_OS\_AllTraded almost simultaneously; other exchanges would only have a return of YD\_OS\_Canceled or YD\_OS\_AllTraded.

### notifyFailedCancelOrder

    virtual void notifyFailedCancelOrder(const YDFailedCancelOrder *pFailedCancelOrder,const YDExchange *pExchange,const YDAccount *pAccount)

Generate notifyFailedCancelOrder when cancel order is rejected.

### notifyTrade

    virtual void notifyTrade(const YDTrade *pTrade, const YDInstrument *pInstrument, const YDAccount *pAccount)

Receive matched trade returns.

### notifyQuote

    virtual void notifyQuote(const YDQuote *pQuote, const YDInstrument *pInstrument, const YDAccount *pAccount)

Receive quotation returns.

### notifyFailedQuote

    virtual void notifyFailedQuote(const YDInputQuote *pFailedQuote, const YDInstrument *pInstrument, const YDAccount *pAccount)

Receive wrong quotation returns.

### notifyFailedCancelQuote

    virtual void notifyFailedCancelQuote(const YDFailedCancelQuote *pFailedCancelQuote,const YDExchange *pExchange,const YDAccount *pAccount)

Receive wrong quotation returns.

### notifyRequestForQuote

    virtual void notifyRequestForQuote(const YDRequestForQuote *pRequestForQuote,const YDInstrument *pInstrument)

Receive request for quote.

### notifyCombPositionOrder & notifyFailedCombPositionOrder

    
    virtual void notifyCombPositionOrder(const YDOrder *pOrder,const YDCombPositionDef *pCombPositionDef,const YDAccount *pAccount)
    virtual void notifyFailedCombPositionOrder(const YDInputOrder *pFailedOrder,const YDCombPositionDef *pCombPositionDef,const YDAccount *pAccount)

Receive YDApi::insertCombPositionOrder, YDExtendedAPI::checkAndInsertCombPositionOrder's return.

If the exchange accepts the user's combined position order, only one return will be sent back via notifyCombPositionOrder. In that, OrderStatus equals YD\_OS\_AllTraded, TradeVolume must equals OrderVolume and OrderSysID must be given.

If the user's combined position order is rejected by ydServer, the return will be sent back via notifyFailedCombPositionOrder. In that, ErrorNo is the error code of ydServer.

If the exchange rejects the user's combined position order, only one return will be sent back via notifyCombPositionOrder. In that, ErrorNo equals the error code of exchange plus 1000, OrderStatus equals YD\_OS\_Rejected, and OrderSyID will not be given.

### notifyOptionExecTogetherOrder & notifyFailedOptionExecTogetherOrder

    
    virtual void notifyOptionExecTogetherOrder(const YDOrder *pOrder,const YDInstrument *pInstrument, const YDInstrument *pInstrument2,const YDAccount *pAccount)virtual void notifyFailedOptionExecTogetherOrder(const YDInputOrder *pFailedOrder,const YDInstrument *pInstrument,const YDInstrument *pInstrument2,const YDAccount *pAccount)

Receive YDApi::insertOptionExecTogetherOrder return.

Suppose users' option trading combination execution is accepted or rejected by the exchange. In that case, the return will be sent back via notifyOptionExecTogetherOrder.

If users' option trading combination execution is rejected by ydServer, the return will be sent back via notifyFailedOptionExecTogetherOrder. In that, ErrorNo equals the error code of ydServer.

### notifyMarketData

    virtual void notifyMarketData(const YDMarketData *pMarketData);

Returns market data.

### notifyAccount

    virtual void notifyAccount(const YDAccount *pAccount);

Notify when changes have been made in account information. By far, if MaxMoneyUsage, Deposit, Withdraw, FrozenWithdraw, TradingRight or LoginCount is modified in YDAccount, notifyAccount will be triggered.

### notify AccountXXXInfo

    virtual void notifyAccountExchangeInfo(const YDAccountExchangeInfo *pAccountExchangeInfo);
    virtual void notifyAccountProductInfo(const YDAccountProductInfo *pAccountProductInfo);
    virtual void notifyAccountInstrumentInfo(const YDAccountInstrumentInfo *pAccountInstrumentInfo);
    

Returns information about trading permission.

### notifyIDFromExchange

    virtual void notifyIDFromExchange(const YDIDFromExchange *pIDFromExchange,const YDExchange *pExchange);

Returns Exchange's system ID.

### notifyUpdateMarginRate

    virtual void notifyUpdateMarginRate(const YDUpdateMarginRate *pUpdateMarginRate);

Returns the response of changing password. If errorNo equals 0, it means passwords changed successfully; otherwise, failed to change password. The administrator can change the user's password.

### notifyChangePassword

    virtual void notifyChangePassword(int errorNo);

Returns the response of changing password. If errorNo equals 0, it means passwords changed successfully; otherwise, failed to change password. The administrator can change the user's password.

### notifyResponse

    virtual void notifyResponse(int errorNo,int requestType);

Returns error ID and request type which causes this error.

### notifyRecalcTime

    virtual void notifyRecalctime(void);

Get recalculation time. It is the suggested time to calculate the margin and the profit or loss of the position while using YDApi; it is the callback every time calculating margin and profit or loss of the position automatically while using YDExtendedApi.

YDApi
-----

YDApi provides three methods for ydClient to connect.

*   Starting ydApi
*   Connecting with ydServer and submiting orders
*   Searching initalized database

### start

    virtual bool start(YDListener *pListener)=0;

\*pListener is a pointer of YDListener. Return false if the configuration file is not set up correctly.

This function achieves making YDApi and YDListener connected to share YDApi's initialized database and internal managing service. It starts internal managing threads YDListener required to monitor network communication according to the configuration file. Theoretically, ydApi supports multiple YDApi/YDListener instances (even two YDApi instances sharing one YDListener instance) within one client program. However, we do not suggest such a development model because of the complexity of its internal data synchronization logic.

### startDestroy

    virtual void startDestroy(void)=0;

After calling this function, api will try to shut down all connections and stops all threads of executions that belong to this api.

During the destruction, all trading functions and notifications are disabled. The data retrieving function is still abled. When destroying is done, notifyBeforeApiDestroy will be sent to the listener. The second phase will start to release all data structures from api.

Never delete api by yourself, but users suppose to delete the listener on their own.

### disconnect

    virtual void disconnect(void)=0;

Disconnect from ydServer.

### login

    virtual bool login(const char *username, const char *password const char *appID, const char *authCode)=0;

Send login request to ydServer. Return false if network disconnects. ydServer's feedback will be retrieved by callback function YDListener::notifyLogin.

ydServer will generate a unique instance number whenever it starts. ydApi will save this instance number whenever ydClient logs in. Successful login after disconnection will receive ydServer's instance number in the login response. If the new instance number doesn't match with the old instance number, ydClient will quit. Since the user's serial sequence will change after a restart, ydClient needs to re-login and synchronize all data to finish the initialization. ydClient needs to monitor its application's status. Once ydServer restarts, ydClient need to decide whether to restart the application or not. If it restarts, ydClient need to synchronize all data and complete initialization.

#### App ID and Authorization Code

Send login request to ydServer. Return false if network disconnects. ydServer's feedback will be retrieved by YDListener::notifyLogin. App ID and authorization code provided below can be used during the development environment and testing environment.

    
    //#define APP_ID "ydDev 1.0"#define AUTH_CODE "9ba749b1e1848a32ba67182c5ffee0fe"
    

You need to provide App ID to the future company before the launch. An authorization code will be issued if the App ID is verified. Use new App ID and authorization code for the production environment.

### insertOrder

    virtual bool insertOrder(YDInputOrder *pInputOrder,const YDInstrument *pInstrument, const YDAccount *pAccount=0)=0;

Send login request to ydServer. Return false if network disconnects. ydServer's feedback will be retrieved by YDListener::notifyOrder or YDListener::notifyFailedOrder. YDListener::notifyOrder handles order returns that are accepted by ydServer but might be rejected by exchanges. Thus, YDListener::notifyOrder needs to determine order returns' ErrorNo.

YDListener::notifyFailedOrder handles order returns that are rejected by ydServer, mainly caused by ydServer's risk management. For failed orders with causes like disconnection and flow control, they are handled the same as those rejected by exchanges.

Due to seat disconnection or flow control, it might be failed to submit or cancel orders. ydServer will return false to ydClient. For failed orders, its OrderStatus will be YD\_OS\_Rejected, and ErrorNo will be given.

Note that this function has two parameters, one is order record, and another is a pointer which points to YDInstrument. It is apparently different from any other trading systems on the market. It reduces copy and comparison of strings to optimize the system's efficiency. Thus, for instruments required by the business, we suggest that after getInstrumentByID() is called, you should cache its pointer to reduce the comparison of strings.

#### Order's status and process

When an order is sent into the ydServer, if it doesn't pass risk management analysis or flow control check, its status will be changed into YD\_OS\_Rejected. ydClient will receive the return from YDListener::notifyFailedOrder. If ydServer restarts after a malfunction, this order's record won't be saved.

If an order passes the risk management analysis or flow control check, its status will be changed into YD\_OS\_Accepted. (This status won't be sent to ydClient.) ydServer will submit this order to exchange and wait for the order return.

After receiving returns from the exchange, if the order passes the exchange's examination, it will become a valid order. Its status will turn into YD\_OS\_Queueing (This order is confirmed by the exchange and saved in ydServer's permanent transaction record. It won't be lost even if ydServer restarts due to a malfunction.) ydClient will receive the callback from YDListener::notifyOrder. If the order does not pass the exchange's examination, its status will turn into YD\_OS\_Rejected. ydClient will receive the callback from YDListener::notifyOrder with YD\_OS\_Rejected status. If ydServer restarts due to a malfunction, the order's record will not be saved.

The order with YD\_OS\_Queueing status might be completely traded, partly traded, or completely untraded. The trading situation can be determined by the field YDOrder::TradeVolume of YDListener::notifyOrder's callback.

If there is a matched trade while queueing, users will receive the callback from YDListener::notifyOrder and YDListener::notifyTrade. If there are multiple matched trades while queueing, users will receive multiple callbacks. YDOrder::TradeVolume will increase as matched trades increase. To illustrate, if an affirmed order with three lots is in the exchange's queue. At first, YDOrder::TradeVolume of YDListener::nofityOrder will be 0. Then, it is divided into three transactions in one minute, one lot per transaction. Therefore, YDOrder::TradeVolume will be 1, 2, 3 in the next three callbacks.

#### Trading Permissions

YD system allows accounts to trade by default. It means that without setting YD\_TR\_CloseOnly, YD\_TR\_Forbidden, and even YD\_TR\_Allow, ydServer will enable accounts to trade regardlessly. No matter how users set the setting, ydServer always allow users to delete their orders. ydServer check users' permission in order of account, instrument, product and exchange. If any of them forbids trading, users can not submit the order; if any prohibits opening positions, users can only liquidate positions.

#### OrderRef

For every user under YD System, OrderRef. Whenever users login in, the maximum OrderRef of all orders and quotations will be retrieved. If a user hasn't submitted any order yet, OrderRef would be 0. We suggest users' automated trading system always number the orders and quotations successively, starting from the next number after MaxOrderRef.

For orders and quotations, all data returned will contain OrdeRef, which users fill when submitting. YD System won't check OrderRef's uniqueness or monotonicity. For orders derived by quotations, the order's OrderRef will be the same as the quotation's. For orders and quotations not submitted by YD System or are not submitted from this time, their OrderRef will be -1; for orders and quotations from YDClient.exe, OrderRef will be 0.

#### Connected Seat Number

ydClient can use YDAPI's getExchange or getExchangeByID methods to retrieve YDExchange's pointer. The ConnectionCount indicates the number of seat connection. Note that connected seat number starts from 0, in ascending order, until ConnectionCount equals to -1.

Exchanges require that all counter systems connect to the exchange via FENS. However, the connected front-end processors(FEP) allocated by FENS are random. Different ydServer instances' same seat might connect to different FEP; even during runtime, a seat might connect to different FEP due to reconnection.

#### Connection Selection

YD\_CS\_Any means not selecting a fixed seat to connect. ydServer submits orders or cancels orders to the exchange as fast as possible. If all connections are unavailable due to flow control or network disconnection, ydServer will retrieve the error report via YDListener::notifyOrder.

YD\_CS\_Fixed means selecting a fixed seat to connect. If the fixed seat is unavailable due to flow control or network disconnection, ydServer will immediately return the error report. If the fixed seat is processing other orders in the meantime, the system will wait for 200 ms. If excess 200 ms, YDListener::notifyOrder will return the error report.

YD\_CS\_Prefered means selected a preferred seat to connect. Suppose the fixed seat is unavailable due to flow control, network disconnection or processing other orders. In that case, it will switch the selection type to YD\_CS\_Any.

#### Error Code

ErrorNo is not required at submitting an order; thus, it is set as 0.

### cancelOrder

    virtual bool cancelOrder(YDCancelOrder *pCancelOrder,const YDExchange *pExchange, const YDAccount *pAccount=0)=0;

Send a cancel order to ydServer. Return false if network disconnects. ydServer's feedback is retrieved by YDListener::notifyOrder. Cancellation is successful if the order's status turns into YD\_OS\_Canceled; the cancellation is failed if the order's status is not YD\_OS\_Canceled or no feedback is retrieved. You should set a time-out to resubmit the cancel order after a period elapsed.

Note, correct cancel order's return will be retrieved by YDListener::notifyOrder, and incorrect cancel order's return will be retrieved by YDListener::notifyFailedCancelOrder.

### insertQuote

    virtual bool insertQuote(YDInputQuote *pInputQuote,const YDInstrument *pInstrument, const YDAccount *pAccount=NULL)=0;

Sent a quotation to ydServer. Return false if network disconnects.

If a quotation is successful, there will be one YDListener::notifyQuote callback and several YDListener::notifyOrder callbacks:

*   YDListener::notifyQuote must include QuoteSysID. Bid order must include BidOrderSysID; Ask order must include AskOrderSysID.
*   There might be more than one YDListener::notifyOrder callbacks. Every derived order will be given separately. The submitted order return, which is sent back by notifyOrder, its YDOrderFlag must be YD\_YOF\_QuoteDerived. The derived orders are allowed to cancel. Derived order's OrderRef is the same as quotation's. The assortment of nofyOrder and notifyQuote is not guaranteed.

There are two conditions if a quotation is failed:

*   If the quotation doesn't pass ydServer's risk management analysis, the quotation return will be retrieved by YDListener::notifyFailedQuote callback. Every quotation only has one callback; ErrorNo states the error code.
*   If exchanges reject the quotation, the quotation return will be retrieved by YDListener::notifyQuote callback. ErrorNo equals the error code of exchange plus 1000. QuoteSysID will not be given.

### cancelQuote

    virtual bool cancelQuote(YDCancelQuote *pCancelQuote,const YDExchange *pExchange, const YDAccount *pAccount=NULL)=0;

Send a cancel quotation to ydServer. Return false if network disconnects.

ydServer's feedback is retrieved by YDListener::notifyOrder. One cancel quotation for one pending quotation. The cancellation is successful If the quotation's status turns into YD\_OS\_Canceled; the cancellation is failed if the quotation's status does not turn into YD\_OS\_Canceled or no feedback retrieved.

### insertCombPositionOrder

    virtual bool insertCombPositionOrder(YDInputOrder *pInputOrder,const YDCombPositionDef *pCombPositionDef,const YDAccount *pAccount=NULL)=0;

Make a combined position or split a combined position. Return false if network disconnects.

If an order is rejected by ydServer, order return will be retrieved by YDListener::notifyFailedCombPositionOrder. If an order is forwarded to the exchange, order return will be retrieved by YDListener::notifyCombPositionOrder.

### insertOptionExecTogetherOrder

    virtual bool insertOptionExecTogetherOrder(YDInputOrder *pInputOrder,const YDInstrument *pInstrument,const YDInstrument *pInstrument2,const YDAccount *pAccount)=0;

Send an option trading combination execution order. Return false if network disconnects.

If an order is rejected by ydServer, order return will be retrieved by YDListener::notifyFailedOptionExecTogetherOrder. If an order is forwarded to the exchange, an order return will be retrieved by YDListener::notifyOpentExecTogetherOrder.

### insertMultiOrders

    virtual bool insertMultiOrders(unsigned count, YDInputOrder inputOrders[],const YDInstrument *instruments[], const YDAccount *pAccount=NULL);

The variable count is the number of orders in a batch, up to 16. Orders sent in a batch can also be cancelled in a batch or be cancelled individually.

Return true if all orders in a batch are accepted; return false if orders are partly or entirely rejected.

### cancelMultiOrders

    virtual bool cancelMultiOrders(unsigned count, YDCancelOrder cancelOrders[],const YDExchange *exchanges[], const YDAccount *pAccount=NULL);

The variable count is the number of cancel orders in a batch, up to 16.

Return true if all cancel orders are accepted; return false if cancellation orders are partly or entirely rejected.

### insertMultiQuotes

    virtual bool insertMultiQuotes(unsigned count, YDInputQuote inputQuotes[],const YDInstrument *instruments[], const YDAccount *pAccount=NULL)=0;

The variable count is the number of quotations in a batch, up to 16. The quotation sent in a batch can also be cancelled in a batch or be cancelled individually.

Return true if all quotaitons in a batch are accepted; return false if quotations are partly or entirely rejected.

### cancelMultiQuotes

    virtual bool cancelMultiQuotes(unsigned count, YDCancelQuote cancelQuotes[],const YDExchange *exchanges[], const YDAccount *pAccount=NULL)=0;

The variable count is the number of cancel quotations in a batch, up to 16.

Return true if all cancel quotations in a batch are accepted; return false if quotations are partly or entirely rejected.

### subscribe

    virtual bool subscribe(const YDInstrument *pInstrument)=0;

According to ydServer's configuration file, ydServer can subscribe to exchange's market data and send them to the client via TCP or UDP.

After the client's initialization (YDListener::notifyFinishInit), YDApi::subscribe can be used to subscribe to designated instruments' market data. It can subscribe to multiple instruments by setting up multiple instruments. If the network disconnects while the client is running, there is no need for users to re-subscribe once reconnected to the network. The earlier subscription is still valid and will receive the latest market data automatically.

After subscribing, market data will be retrieved via YDListener::notifyMarketData() method. Suppose TCP market data and UDP market data are both configured in the configuration file. In that case, ydApi will merge those market data to ensure one notice for one market data client.

Return false if network disconnects.

Historical market data subscription is not supported.

### unsubscribe

    virtual bool unsubscribe(const YDInstrument *pInstrument)=0;

Send request for unsubscribing to ydApi/ydServer. Return false if network disconnects. No callback will be received from YDListerner::notifyMarket if users unsubscribe.

### setTradingRight

    virtual bool setTradingRight(const YDAccount *pAccount, const YDInstrument *pInstrument, const YDProduct *pProduct, const YDExchange *pExchange, int tradingRight)=0;

An administrator interface.

If pInstrument is given to set the account's trading right of a specific instrument, pProduct and pExchange are meaningless and should be filled with NULL. If pProduct is given to set the account's trading right of a specific product, pInstrument and pExchange are meaningless and should be filled with NULL. If pExchange is given to set the account's trading right in a specific exchange, pInstrument and pProduct are meaningless and should be filled with NULL.

YD system allows accounts to trade by default. It means that without setting YD\_TR\_CloseOnly, YD\_TR\_Forbidden, and even YD\_TR\_Allow, ydServer will enable accounts to trade regardlessly. No matter how users set the setting, ydServer always allow users to delete their orders. ydServer check users' permission in order of account, instrument, product and exchange. If any of them forbids trading, users can not submit the order; if any prohibits opening positions, users can only liquidate positions.

Return false if network disconnects. ydServer's feedback is retrieved by YDListener::notifyAccount. notifyAccount indicates the account level; notifyAccountExchangeInfo indicates the exchange level; notifyAccountProductInfo indicates the product level; notifyAccountInstrumentInfo indicates the instrument level. If the account you are trying to modify is connecting to ydServer, notification will also be received.

### alterMoney

    virtual bool alterMoney(const YDAccount *pAccount, int alterMoneyType, double alterValue)=0;

An administrator interface used for setting balance.

Return false if network disconnects. ydServer's feedback will be retrieved by the callback function YDListener::notifyAccount. If the account you are trying to modify is connecting to ydServer, notification will also be received.

### updateMarginRate

    virtual bool updateMarginRate(const YDUpdateMarginRate *pUpdateMarginRate)=0;

An administrator interface used for setting margin rate.

Return false if network disconnects. ydServer's feedback will be retrieved by the callback function YDLisener::notifyUpdateMarginRate. If the account you are trying to modify is connecting to ydServer, notification will also be received.

### changePassword

    virtual bool changePassword(const char *username, const char *oldPassword, const char *newPassword)=0;

Change user's password. Standard users are allowed to change their own password. Administrators are allowed to change their own password and standard users' password. When an administrator changes a standard user's password, oldPassword will be set as NULL.

Return false if network disconnects. ydServer's feedback will be retrieved by the callback function YDLisener::notifyChangePassword's parameter errorNo.

### selectConnections

    virtual bool selectConnections(const YDExchange *pExchange,unsigned long long connectionList)=0;

Send the connection selection result to ydServer. When ydServer receives orders using YD\_CS\_Any, it will send order based on connection selection result.

This function can only be used by administrators or accounts with YD\_AF\_SelectConnection indicator. connectionList is a list which occupies 4 bytes, smallest number has a highest piority, must cover all connections needs to switch.

For example, if need to set the connection order as 2-3-0-1-4 that connectionList will be expressed in binary as 0100 0001 0000 0011 0010.

### hasFinishedInit

    virtual bool hasFinishedInit(void)=0;

Determine whether ydApi receives complete initialized data from ydServer. If true, data initializaiton is finished that initialized data and user-defined fields can be used securely.

### getSystemParam

    virtual int getSystemParamCount(void);virtual const YDSystemParam *getSystemParam(int pos);virtual const YDSystemParam *getSystemParamByName(const char *name,const char *target);

Get system's parameter.

### getExchange

    virtual int getExchangeCount(void)=0;virtual const YDExchange *getExchange(int pos)=0;virtual const YDExchange *getExchangeByID(const char *exchangeID)=0;

Get exchange's basic information, including seat connections, risk management and flow control.

### getProduct

    virtual int getProductCount(void)=0;virtual const YDProduct *getProduct(int pos)=0;virtual const YDProduct *getProductByID(const char *productID)=0;

Get product's information

### getInstrument

    virtual int getInstrumentCount(void)=0;virtual const YDInstrument *getInstrument(int pos)=0;virtual const YDInstrument *getInstrumentByID(const char *instrumentID)=0;

Get instrument's information

### getCombPositionDef

    virtual int getCombPositionDefCount(void)=0;virtual const YDCombPositionDef *getCombPositionDef(int pos)=0;virtual const YDCombPositionDef *getCombPositionDefByID(const char *combPositionID,int combHedgeFlag)=0;

Get combined position definition's information.

### getAccount

    virtual int getAccountCount(void)=0;virtual const YDAccount *getAccount(int pos)=0;virtual const YDAccount *getAccountByID(const char *accountID)=0;

For administrators only. Get account's information.

Standard user should use function below to acquire account's information

### getPrePosition

    //following function for YDAccount can only be used by tradervirtual; const YDAccount *getMyAccount(void)=0;

Get the value of position holding before the beginning of the day. Static data and won't change within the same trading day.

### getMarginRate

    virtual int getMarginRateCount(void)=0;virtual const YDMarginRate *getMarginRate(int pos)=0;

Get the value of the margin rate. Due to the hierarchical structure, the value is usually sparse. In general, you should use getAccountInstrumentInfo to get the specific instrument's margin rate.

### getCommissionRate

    virtual int getCommissionRateCount(void)=0;virtual const YDCommissionRate *getCommissionRate(int pos)=0;

Get the value of the commission rate. Due to the hierarchical structure, the value is usually sparse. In general, you should use getAccountInstrumentInfo to get the specific instrument's commission rate.

### getAccountInfo

    virtual const YDAccountExchangeInfo *getAccountExchangeInfo(const YDExchange *pExchange, const YDAccount *pAccount=NULL)=0;virtual const YDAccountProductInfo *getAccountProductInfo(const YDProduct *pProduct, const YDAccount *pAccount=NULL)=0;virtual const YDAccountInstrumentInfo *getAccountInstrumentInfo(const YDInstrument *pInstrument, const YDAccount *pAccount=NULL)=0;

Get information about the user's exchange, product, instrument, margin rate and commission rate. Those functions ensure values at every level. In general, you should use getAccountInstrumentInfo to get the specific value.

### getGeneralRiskParam

    virtual int getGeneralRiskParamCount(void)=0;virtual const YDGeneralRiskParam *getGeneralRiskParam(int pos)=0;

Get general risk management parameters.

### writeLog

    virtual void writeLog(const char *format,...)=0;

If there is ./log under yd.so(Linux) or yd.dll(Windows) directory, ydApi will save key log about network communication. The log applies syslog format and is named ccyymmddlog.txt. ydClient can use the writeLog method to modify the log.

### getVersion

    virtual const char *getVersion(void)=0

Get ydApi's four-part version number.

### getClientPacketHeader

    virtual int getClientPacketHeader(YDPacketType type, unsigned char* header, int len)=0;

Get the packet header of the client while using raw protocol orders. In that, YDpacketType can be YD\_CLIENT\_PACKET\_INSERT\_ORDER or YD\_CLIENT\_PACKET\_CANCEL\_ORDER. header and len is the head pointer and the length of storage space pre-created by the user. The storage space is used for storing header data generated by ydApi. len should be greater than the header length of corresponding types. This method can only be called after notifyFinishInit.

The return value is the actual header length. The header length of orders and cancel orders are both 16. Return 0 if failed to acquire the packet header, mainly because of early call time or lack of permission to use raw protocol orders.

### getTradingDay

    virtual int getTradingDay(void)=0;

Get the trading date after successfully logging in.
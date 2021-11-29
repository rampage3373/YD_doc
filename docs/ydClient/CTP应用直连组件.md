为了方便现有CTP应用在不修改或者少修改的前提下直接迁移至易达交易系统，易达推出了符合CTP API接口协议的应用直连组件。Linux平台和Windows平台均有支持。

使用方法
----

### 动态库准备

首先，请使用ydCTP的动态库拷贝至程序目录，并使用ydCTP中配套的头文件。Win32环境的动态库是thostmduserapi\_se.dll和thosttraderapi\_se.dll，Linux环境下动态库是thostmduserapi\_se.so和thosttraderapi\_se.so。

其次，把易达客户端动态库拷贝到运行目录下，Win32环境的动态库是yd.dll，Linux环境下动态库是yd.so。

### 程序中的注意点

使用ydCTP的客户程序逻辑不需要修改，但请在以下两个地方做适当修改。

首先，调用CTP交易和行情API的RegisterFront()时，请使用易达柜台的地址和端口，其中交易和行情的端口请统一使用易达交易端口。例如，我们的投资者互联网环境中提供了某环境的交易端口为40400，行情端口为40401，那么在交易和行情API中调用RegisterFront()时填写的端口号均为40400。

其次，调用CTP交易API的ReqAuthenticate()时，请使用易达的AppID和AppAuthCode，由于CTP接受的AppAuthCode的最大长度为16位，如果你在易达中生成的AppAuthCode的长度大于16位的，请在ReqAuthenticate()中仅填写前16位易达的AppAuthCode即可。

然后，本CTP接口中，错误的ErrorID始终为1，易达的错误码放在ErrorMsg之中。

最后，如需指定除了易达交易服务器地址和端口之外的参数，请在客户程序运行目录下创建一个名为“YDClient.ini”的文本文件，并把参数定义写在这个文件中，格式参见ydAPI文档。

性能开销
----

在一个典型的测试环境下，客户程序通过易达CTP接口库报单要比直接通过ydAPI报单多花150-200ns的时间。由于不同运行环境测试条件差异巨大，这个数字仅供参考。

快期安装配置样例
--------

快期是常用的基于CTP接口的桌面应用，通过演示快期系统结合本组件的实际例子，便于用户直观掌握本组件的使用方法。在生产环境中通过快期直连易达交易系统，可以在更加熟悉的桌面环境中使用易达系统。

需要说明的是，由于从快期3.0开始，其行情服务会连接自己的服务器，不受配置的影响。因此，为了能演示连接易达行情时的效果，本文以快期2.93版本为例。

首先，安装快期并更新，然后找到快期执行和配置文件所在目录，注意这个目录通常不是安装快期时指定的安装目录，而是一般在：C:\\Users\\<Windows 用户名>\\AppData\\Roaming\\Q7\*\_standard\\<日期>目录。该路径最后的<日期>可能有多个，日期最新的是当前版本，目录中有broker.xml和别的文件。

把这个目录复制到自己的工作目录准备修改，以免影响原先连接CTP的快期的使用。下面的叙述中用“快期目录”指复制过来的快期当前版本目录。

在快期目录中，编辑broker.xml文件，将其中Trading和MarketData的URL都改成ydServer的地址和端口，broker.xml中通常有多个服务器的配置，其余的服务器配置可以删除。

    <Server>
        <Name>YD-CFFEX(看穿)</Name>
        <Se>1</Se>
        <Trading>
             <item>192.168.15.23:40100</item>
        </Trading>
        <MarketData>
             <item>192.168.15.23:40100</item>
    </MarketData>
    </Server>

把YD\_CTP发布包lib\\win32\\目录下的2个dll文件复制到快期目录中，覆盖原有文件。

把YD\_CTP发布包“example\\”目录下的"YDClient.ini"复制到快期目录中。

请期货公司把快期在该期货公司的AppID和AppAuthCode添加到ydServer的配置文件config/AppAuthCode.csv中。

运行快期目录中的"q7\_release.exe"或者"shinny\_client.exe", 用ydServer的用户名密码登录。

限制
--

尚未支持的CTP交易API函数：

    		ReqUserAuthMethod()
    		ReqGenUserCaptcha()
    		ReqGenUserText()
    		ReqUserLoginWithCaptcha()
    		ReqUserLoginWithText()
    		ReqUserLoginWithOTP()
    		ReqParkedOrderInsert()
    		ReqParkedOrderAction()
    		ReqQueryMaxOrderVolume()
    		ReqRemoveParkedOrder()
    		ReqRemoveParkedOrderAction()
    		ReqBatchOrderAction()
    		ReqOptionSelfCloseInsert()
    		ReqOptionSelfCloseAction()
    		ReqCombActionInsert()
    		ReqQryTradingCode()
    		ReqQryExchange()
    		ReqQryTransferBank()
    		ReqQryCFMMCTradingAccountKey()
    		ReqQryEWarrantOffset()
    		ReqQryInvestorProductGroupMargin()
    		ReqQryExchangeMarginRate()
    		ReqQryExchangeMarginRateAdjust()
    		ReqQryExchangeRate()
    		ReqQrySecAgentACIDMap()
    		ReqQryProductExchRate()
    		ReqQryMMInstrumentCommissionRate()
    		ReqQryMMOptionInstrCommRate()
    		ReqQryInstrumentOrderCommRate()
    		ReqQrySecAgentTradingAccount()
    		ReqQrySecAgentCheckMode()
    		ReqQrySecAgentTradeInfo()
    		ReqQryOptionSelfClose()
    		ReqQryInvestUnit()
    		ReqQryCombInstrumentGuard()
    		ReqQryCombAction()
    		ReqQryParkedOrder()
    		ReqQryParkedOrderAction()
    		ReqQryBrokerTradingAlgos()

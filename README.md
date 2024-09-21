
\-\-
　　痞子衡维护的 NXP\-MCUBootUtility 工具距离上一个大版本(v5\.3\.0\)发布过去一年了，期间痞子衡也做过三个版本更新，但不足以单独介绍。这一次痞子衡为大家带来了全新重要版本v6\.3\.x，这次更新主要是想和大家特别聊聊 ROM 启动日志这个特性的支持。


### 一、v6\.0 \- v6\.3更新记录



```
-- v5.3.2
Improvement:
    1. [RTyyyy] 使能 RT1180 的 XMCD 支持
Bufixes:
    1. [RTyyyy] XMCD配置界面里write dummy cycle设置不生效

-- v6.0.0
Features:
    1. [Wireless] 支持K32W0x1
    2. [Wireless] 支持RW61x
    3. [RW61x] 支持裸应用程序文件作为输入源文件
    4. [RW61x] 支持UART和USB-HID两种下载方式（COM端口/USB设备自动识别）
    5. [RW61x] 支持用于开发阶段的非安全加密启动（未签名）
    6. [RW61x] 支持FlexSPI NOR启动设备
Bufixes:
    1. [RTyyyy] 修复对第二个FlexSPI NOR设备的下载支持
    2. [RTxxx] 修复对FlexSPI NOR设备的下载支持

-- v6.1.0
Features:
    1. [RTxxx] 支持i.MXRT700 A0
    2. [RT700] 支持裸应用程序文件作为输入源文件
    3. [RT700] 支持UART和USB-HID两种下载方式（COM端口/USB设备自动识别
    4. [RT700] 支持用于开发阶段的非安全加密启动（未签名）
    5. [RT700] 支持XSPI NOR启动设备

-- v6.2.0
Features:
    1. [RT700] 支持下载程序进第二个XSPI上连接的启动设备
Bufixes:
    1. [RT700] 当NOR Flash的FDCB区域非空时，下载可能报错
    2. [RT700] 当待下载程序链接在安全SRAM地址时，下载会报错

-- v6.3.0
Features:
    1. [RT] 可以支持获取并解析ROM启动日志
Improvement:
    1. [RT] 对FlexSPI NOR设备做擦除时，可自定义对齐长度
    2. [RT1170] 增加对英飞凌S28H系列Octal Flash支持

```

### 二、几个不可忽视的更新


#### 2\.1 初步支持RT700


　　i.MX RT700 是 RT 三位数家族最新一代旗舰产品，是 i.MX RT500 的升级，可以说是恩智浦有史以来最强大最复杂的 MCU。鉴于官网还没有发布这颗芯片，这里暂不过多介绍了。


#### 2\.2 支持自定义FlexSPI NOR设备擦除对齐长度


　　我们知道软件对于 RT 四位数的启动设备下载支持，靠得是加载二级 Flashloader 实现的。无论是一键下载模式，还是通用编程器模式，软件都会将擦除范围参数（起始地址，长度）传递给 Flashloader 处理，而 Flashloader 里会自动做对齐处理（根据实际情况，组合 Block 和 Sector 擦除命令，比如粗粒度先用 Block 擦除，细粒度再用 Sector 擦除）。


　　上述 Flashloader 里的关于擦除处理机制看似很完美，但是对于一些特殊类型的 Flash 可能会失效。比如 [Infineon MirrorBit Flash](https://github.com) 类型，这种 Flash 仅在某几个特定 Block 上支持 Sector 擦除，对于这种情况，就不能任由 Flashloader 来管理擦除粒度了，因此需要用户能够强制指定擦除对齐长度。


　　在工具目录 \\src\\targets\\xxx\\bltargetconfig.py 文件中仅可见如下定义，我们可以改变这个定义值来设置擦除对齐长度，对于 Infineon MirrorBit Flash，我们需要将擦除对齐设为 Block 长度 256KB。



```
xspiNorEraseAlignment = 1 # in byte

```

#### 2\.3 对于RT ROM启动日志解析支持


　　i.MX RT 系列发布至今，如果要给客户支持问题类型做一个总结，基本上启动相关问题要占 30%。如果遇到芯片无法启动问题，除了常规经验以外，我们还可以通过 ROM 启动日志来辅助分析。


　　所谓 ROM 启动日志，就是 ROM 在执行过程中记录的状态，这个状态数据被存在在芯片内部 RAM 固定位置处（芯片出厂后，这个位置就无法更改了，被写死在 ROM 代码里）。如果遇到启动失败问题，我们可以读出日志数据予以分析。


　　软件支持两种途径获取并解析启动日志数据：



> * 途径一：用户根据界面里 Log Start， Log Length 信息先读取出日志数据文件(比如用 JLink 去读取)，然后在界面 Log Data 框里选取这个日志文件路径，最后点击 View Boot Log 按钮解析。
> * 途径二：在芯片启动失败自动转入串行下载模式时，不勾选软件 One Step 模式，单步连接保证芯片处于 Flashloader 模式（便于 blhost 工具读取内部 RAM），直接点击 View Boot Log 按钮获取并解析。


　　途径二通过软件直接从芯片 RAM 里读取启动日志数据有一个注意点，因为是借助加载 Flashloader 执行实现的，所以如果 Flashloader 依赖的 RAM 空间与启动日志存储空间有冲突，可能会导致日志数据损坏。


![](https://raw.githubusercontent.com/JayHeng/pzhmcu-picture/master/cnblogs/nxpSecBoot_v6.3.0_parse_log.PNG)


　　至此，这次更新的主要特性便介绍完了。MCUBootUtility项目地址如下。虽然当前版本（v6\.3\.x）功能已经非常完备，你还是可以在此基础上再添加自己想要的功能。如此神器，还不快快去下载试用？



> * 地址1： [https://github.com/nxp\-mcuxpresso/mcu\-boot\-utility](https://github.com)
> * 地址2： [https://github.com/JayHeng/NXP\-MCUBootUtility](https://github.com)
> * 地址3： [https://gitee.com/jayheng/NXP\-MCUBootUtility](https://github.com)


### 欢迎订阅


文章会同时发布到我的 [博客园主页](https://github.com):[樱花宇宙官网](https://yzygzn.com)、[CSDN主页](https://github.com)、[知乎主页](https://github.com)、[微信公众号](https://github.com) 平台上。


微信搜索"**痞子衡嵌入式**"或者扫描下面二维码，就可以在手机上第一时间看了哦。


![](https://raw.githubusercontent.com/JayHeng/pzhmcu-picture/master/wechat/pzhMcu_qrcode_258x258.jpg)



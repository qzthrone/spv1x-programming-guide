.. _jtag-debug-workflow:

调试Flash应用程序
======================

简介
-------------------------------

 SPV1x支持JTAG（4线）在线调试和cJTAG（2线）在线调试。两者的调试体验相近，但是cJTAG需要的引脚更少，因此推荐使用cJTAG调试。
 由于芯片的调试存在诸多限制，因此请务必仔细阅读本章内容后面的注意事项。


准备工作
-------------------------------

 1. JTAG调试器(Straw Mini)安装驱动

  首次使用调试器，需要安装调试器驱动。

  (1). 安装调试器原厂驱动。

   调试器驱动程序安装文件在SDK包内，具体文件为：tool->下载器驱动->FTDI系列->CDM212364_Setup.zip。压缩包解压后，双击exe文件开始安装，勾选同意协议并点击next，直至安装完成。

  (2). 给调试器打libusbk驱动。

   驱动替换工具在SDK包内，具体文件为：tool->驱动替换工具-> zadig-2.4.exe。工具为绿色软件，双击exe即可运行。

   a. 软件打开后，先点击”Options”->“List All Devices”，列举出所有支持的设备。

   .. image:: ../_static/debug-zadig-list-all-devices.png
      :align: center


   b. 在设备下拉列表中，找到”SpaceTouch USB Device(Interface 0)”。

    .. image:: ../_static/debug-zadig-select-device.png
      :align: center


   c. 在”Driver”一行，将目标驱动选择为libusbK。

   .. image:: ../_static/debug-zadig-select-driver.png
      :align: center


   d. 点击”Replace Driver”按钮，开始替换驱动，并耐心等待驱动替换完成。

   .. image:: ../_static/debug-zadig-replace-driver.png
      :align: center


   e. 最后，在系统设备管理器中，确认出现对应的”libusbK USB Devices”。

   .. image:: ../_static/debug-check-libusbk-device.png
      :align: center


 .. note::

  Straw Mini采用FT2232HL作为主控芯片，该IC具有2路信号通道。JTAG/cJTAG调试只用到通道0，通道1可以用作SPV1x的下载和调试打印串口。
  如果电脑没有出现通道1的串口，请到设备管理器中检查” USB Serial Converter B”是否勾选了”加载VCP”选项，如下图所示：

  .. image:: ../_static/debug-enable-ch1-vcp-driver.png
      :align: center


 2.	复制OpenOCD配置文件到OpenOCD程序目录下

  OpenOCD配置文件在SDK包内，具体位置为：tool->openocd配置文件->openocd_spv1x_jtag.cfg和tool->openocd配置文件->openocd_spv1x_cjtag.cfg，这2个文件分别对应JTAG调试和cJTAG调试

 3. 下载需要调试的程序到芯片

  编译需要调试的工程，并将生成的bin文件，使用串口下载到芯片。后续如果代码发生更改，则需要重新下载更改后的程序，以保证代码和芯片内部的bin文件一致。

 .. note::

  在执行用户程序时，芯片GPIO引脚的JTAG复用功能默认是关闭的。为了保证程序运行过程中，调试器可以连接到芯片的JTAG，需要在程序中显式地开启GPIO引脚的JTAG复用功能。
  
  对于JTAG调试，需要在app_init()的最前面加上如下代码：

  .. code-block:: c

    DEV_GPIO->CTL[0] = 10;  //CPU_TCK
    DEV_GPIO->CTL[1] = 10;  //CPU_TMS
    DEV_GPIO->CTL[2] = 10;  //CPU_TDO
    DEV_GPIO->CTL[3] = 10;  //CPU_TDI

  对于cJTAG调试，需要在app_init()的最前面加上如下代码：

  .. code-block:: c

    DEV_GPIO->CTL[0] = 10;  //CPU_TCK
    DEV_GPIO->CTL[1] = 10;  //CPU_TMS


 4. 连接调试器和待调试的芯片。

  (1) 如果使用JTAG调试，则需要连接6根线：

    a. 调试器TCK连接芯片GPIO00;
    b. 调试器TDO连接芯片GPIO02;
    c. 调试器TMS连接芯片GPIO01;
    d. 调试器TDI连接芯片GPIO03;
    e. 调试器GND连接芯片GND;
    f. 调试器VREF连接芯片IOVCC;

  (2). 如果使用cJTAG调试，则需要连接4根线：

    a. 调试器TCK连接芯片GPIO00;
    b. 调试器TMS连接芯片GPIO01;
    c. 调试器GND连接芯片GND;
    d. 调试器VREF连接芯片IOVCC;

 5. IDE调试功能配置

  工程的调试功能配置，只需要在首次启动调试前配置一次，后续启动调试不用再配置。

  (1). 对需要进行调试的工程右键，然后选择“Debug As”->”Debug Configurations…”。

  .. image:: ../_static/debug-open-configurations-dialog.png
      :align: center

  (2). 在弹出的”Debug Configurations”窗口中，双击”GDB OpenOCD Debugging”，新建一个调试配置。

  .. image:: ../_static/debug-create-configuration.png
      :align: center

  (3). 在”Debug Configurations”窗口的”Main”标签页中，选择需要调试的”C/C++ Application”。

   a. 点击"Browse..."按钮

   .. image:: ../_static/debug-select-application-step1.png
    :align: center

   b. 在弹出的文件选择对话框中，选择需要调试的elf程序文件

   .. image:: ../_static/debug-select-application-step2.png
    :align: center

   c. 选择完成后的效果

   .. image:: ../_static/debug-select-application-step3.png
    :align: center

  (4). 在”Debug Configurations”窗口的”Debugger”标签页中，配置openocd和gdb相关配置，参考配置如下：

  .. image:: ../_static/debug-configuration-table-debugger.png
      :align: center

  配置部分文本内容：

  .. code-block:: c

    ${openocd_path}/${openocd_executable}

    -f "${openocd_path}/openocd_spv1x_cjtag.cfg"

    ${cross_prefix}gdb${cross_suffix}

    set  remotetimeout 250
    set arch riscv:rv32

  .. note::

   OpenOCD的 “Actual executable”中的内容，与IDE的安装路径相关，不同电脑显示的路径会不一样。
   如果路径存在错误，需要在"Window"->"Preferences"->"MCU"->"Global OpenOCD Path"中进行配置。如下图所示：

   .. image:: ../_static/debug-global-openocd-path.png
      :align: center

  (5). 在”Debug Configurations”窗口的”Startup”标签页中，配置调试启动时的一些初始化参数和行为，参考配置如下：

  .. image:: ../_static/debug-configuration-table-startup.png
      :align: center
 
  配置部分文本内容：

  .. code-block:: c

    mem 0x48000000 0x48FFFFFF ro
    mem 0x4C000000 0x4CFFFFFF ro

    41000000
    app_init

  (6). 点击“Apply”，然后点击”Debug”，启动调试。后续可以直接使用IDE主界面上的快捷图标启动调试。
 
  .. image:: ../_static/debug-start.png
      :align: center

注意事项
-------------------------------

 1. SPV1x只有2个硬件断点。对于c语言程序的调试，由于需要给gdb留一个临时断点，因此用户同一时刻，只能打一个断点。
 2. SDK默认是O1优化，编译器对程序进行优化后，程序的实际执行流程和源代码中的代码顺序会有所出入，但是结果的正确性是可以保证的。
 3. 开启调试前，请确认芯片内部的程序和IDE中的代码是一致的。程序不一致会出现许多让人误解的行为，如IDE显示在执行A函数，但是芯片实际效果是B函数的行为。
 4. IDE在进入调试后，会切换到调试界面布局。可以通过"Window"->"Perspective"->"Open Derspective"->"C/C++"切换回代码编辑界面。或者通过菜单栏的快捷图标进行切换。

 .. image:: ../_static/debug-switch-perspective.png
      :align: center






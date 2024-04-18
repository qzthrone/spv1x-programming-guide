.. _changelog_sdk:

SPV1x SDK 版本迭代记录
======================

.. important::
  
  SDK v1.2.0以上 开始为量产版本，针对大批量基台量产烧录优化，请及时更新。

 :download:`点此浏览和下载过往版本SDK <https://gitee.com/spacetouch-zh/sdk-spv1x/tags>`

1.2.4-rc-01252024
---------------------------

.. warning:: 
  
  基于SDK v1.2.3的用户工程升级，需要完成以下适配

 1. 将app_init()中通过vmic_iovcc_adjust()对iovcc/vmic进行设置的流程删除。
 2. 在app_init()中调用sys_set_iovcc()对iovcc电压进行静态设置。
 3. 如果使用audio adc和(或)gpadc，需要在app_init()中调用adc_sys_init()对adc进行初始化。如果只使用gpadc，adc_sys_init()的“采样率”和“增益”参数设置无影响。
 4. 删除其余流程中可能存在的vmic_iovcc_adjust()调用，例如存在于tick_handler_100ms()中的iovcc/vmic动态调整逻辑。

.. versionadded:: 1.2.4
     
     1. lowlevel静态库 API：
   
      - adc_sys_init() ADC初始化（同时设置vmic电压）；
      - sys_set_iovcc() 调整iovcc电压；
      - sys_set_vmic() 调整vmic电压；
      - sys_get_vcc() 读取vcc电压。
  
     2. sop8封装spv100a4对应的链接器脚本。
     3. sop16封装spv1x1a4对应的链接器脚本。
     4. gpio API: gpio_set_mfp() 配置io口mfp功能。
     5. soc API: soc_watchdog_start/stop/clear() 控制看门狗定时器, soc_set_brom_flag() 配置brom预设功能标志位。
     6. 16KHz MP3播放器库 API: player_mp3_16khz_set_loop(), 用于设置正在播放的音乐的单曲循环标志。

.. versionchanged:: 1.2.4
     
     1. 16KHz MP3播放器库：调整内部DRC参数，优化音量体验。
     2. s1a播放器库：调整内部DRC参数，优化音量体验。
     3. 全部封装的链接器脚本(.ld): 进一步修正flash_op_init()调用在链接阶段可能触发的relocation报错问题。
     4. start.S为Header Section追加新信息，便于后续项目开展。
     5. 音频转换和打包工具升级到v1.2.0，优化s1a编码器效果，为mp3编码器提供平均码率(abr)选项。

.. deprecated:: 1.2.4

     1. lowlevel静态库：

      - 根据vcc电压对iovcc和vmic电压进行动态调整的机制作废，用户需要根据方案实际的供电情况，手动设置固定的iovcc和vmic电压。
      - 删除API: vmic_iovcc_adjust()。

     2. ssop24封装spv1x2a8链接器脚本作废。

1.2.3-rc-12152023
---------------------------

.. versionchanged:: 1.2.3
     
     1. 修复flash_op_init()在链接阶段可能触发的relocation错误。
     2. 调整lpm_hibernate_enter()的实现。
     3. 优化demos/spv120a4_asr模版工程的iovcc电压策略。
   
1.2.2-rc-12082023
---------------------------

.. versionadded:: 1.2.2
     
     1. 链接器脚本(.ld)追加ADPCM播放所需的overlay结构
     2. ODT工具升级至 v1.3.0 版本，提供对外挂SPI NOR Flash数据介质的烧录功能。
     3. 提供Force Hibernate API: lpm_hibernate_enter()，确保Hiberate模式的稳定进入。
     4. SDK现开始提供demos文件夹，包含特定SPV1x封装的典型使用场景演示。

.. versionchanged:: 1.2.2
     
     1. 修复templates/edpm工程下的Keypad Matrix API逻辑问题。
     2. 修改IOVCC和VMIC电压的初始化策略：初始电压固定为 IOVCC = 3.2v, VMIC = 3.0v。

.. deprecated:: 1.2.2

     - OSC CORE时钟设定现在仅支持 80/100 MHz 档位， 60/70/90/110 MHz 档位的API枚举定义已删除。

1.2.1-rc-11162023
---------------------------

.. versionadded:: 1.2.1
     
     1. :ref:`audio-effect-lib` 增加参量均衡器(EQ)功能。
     2. 静态库 *lowlevel* 提供适配外接Flash所需的Delay Chain调节功能。

1.2.0-rc-11102023
---------------------------

.. versionchanged:: 1.2.0
     
     1. 优化量产烧录流程的特定改动。
     2. 用户程序初始化阶段, *system_init()* 将复位特定PMU寄存器组合，确保多个ONOFF按键唤醒场景和唤醒后ONOFF键值检测逻辑的正确执行。
     3. SDK/cpu文件夹下现针对SPV1x SoC不同封装提供对应的链接器脚本(.ld)，体现内部/外接NOR Flash单元的正确配置：容量/核心参数(如高速模式下Dummy Cycle)，直接影响该封装下程序的启动和执行，请根据实际情况选择，方法如下：
         
        - 在 :guilabel:`Project Explorer` 下当前工程名点击鼠标右键，在弹出菜单中选择 :guilabel:`Properties`。
        - 如下图所示，:menuselection:`C/C++ Build --> Settings --> GNU RISC-V Cross C Linker --> General --> Script Files`，
          双击当前条目，选择符合实际情况的ld文件。  

        .. image:: _static/kiwi-lds-config.png
         :align: center

        注：原有默认.ld文件现更名为generic_flash.ld，适用于EVB/Demo Board等官方提供的开发套件。
     4. 静态库 *lowlevel* 和启动代码 *start.S* 跟随上述改动做必要修改。
     5. 静态库: :ref:`recorder-lib` 接入双二阶高通滤波器改善低频体验，API层面无变化。
     6. 优化链接器脚本架构，提供更多可用SRAM空间。

.. versionadded:: 1.2.0
     
     1. 发布固件烧录程序ODT v1.2.0版本，与SDK v1.2.0配套使用。

1.1.0-rc-11032023
---------------------------

 .. versionadded:: 1.1.0
     
     - 静态库：音效(Audio Effect)，参见 :ref:`audio-effect-lib`。
     - 执行JTAG调试所需的调试器驱动程序、openocd配置文件等。
     - 驱动半双工模式外置Flash所需的代码（需配套固件烧录程序ODT v1.1.0版本使用）。

 .. versionchanged:: 1.1.0
     
     - 固件烧录程序 ODT：升级至v1.1.0版本，提供对半双工Flash的烧录支持，详情参见 :ref:`firmware-downloader`。 
     - 静态库: :ref:`recorder-lib` 在16KHz采样率下提供可选的编码码率:16、24、32、64kbps。
     - 静态库: :ref:`16KHz-mp3-player` 改善“单曲循环”设置下的播放体验, API层面无需适配。
     - 静态库：flash_api，对半双工模式Flash进行适配, API层面无需适配。
     - 静态库：lowlevel，对半双工模式Flash进行适配, API层面无需适配。

1.0.3-rc-10262023
---------------------------

 .. versionadded:: 1.0.3
     
     - 向SDK/templates下属模版工程“edpm”添加了按键API相关源文件，参见 :ref:`key-module`。

1.0.2-rc-10242023
---------------------------

 .. versionadded:: 1.0.2
     
     - 基于SPV1x EVB的Demo工程： **宠物训练发声器**，参见 :ref:`evb-demos`。

1.0.1-rc-10162023
---------------------------

 .. versionchanged:: 1.0.1
     
     - 修复 **播放器场景库** 中“手动暂停播放后，恢复播放时失败” 的问题。本次改动仅影响kss/sys文件夹内容。

1.0.0-rc-10112023-hotfix
---------------------------

 .. versionchanged:: 1.0.0-rc-hotfix
     
     - 调整 **uart_init()** 函数中串口时钟分频系数配置规则，提高高波特率通讯环境下数据稳定性。
    
      .. code-block:: c

       uint32_t div = (((DEV_CLK_FREQ <<7) / baud + 64) >> 7) - 1;

1.0.0-rc-09262023
------------------------

 .. versionadded:: 1.0.0-rc
    
     - 首次公开发布。

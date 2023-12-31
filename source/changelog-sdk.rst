.. _changelog_sdk:

SPV1x SDK 版本迭代记录
======================

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

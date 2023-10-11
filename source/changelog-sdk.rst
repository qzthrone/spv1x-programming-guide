.. _changelog_sdk:

SPV1x SDK 版本迭代记录
======================

1.0.0-rc-10112023-hotfix
---------------------------

 .. versionchanged:: 此
     
     - 调整 **uart_init()** 函数中串口时钟分频系数配置规则，提高高波特率通讯环境下数据稳定性。
    
      .. code-block:: c

       uint32_t div = (((DEV_CLK_FREQ <<7) / baud + 64) >> 7) - 1;

1.0.0-rc-09262023
------------------------

 .. versionadded:: 此
    
     - 首次公开发布。

针对低功耗场景优化的Nor Flash接入策略
======================================

.. note::
   
 前置知识：

   1. :ref:`pmu-module`
   2. 上电后，GPIO19默认为NORVCC引脚，用于向Nor Flash供电。
   3. DEV_PMU->PWR_CTL_f.NORM_NORVCC_EN用于控制芯片在活动模式（ACTIVE）和睡眠模式（SLEEP）下NORVCC的使能，DEV_PMU->PWR_CTL_f.STBY_NORVCC_EN用于控制芯片在待机模式（STANBY）下NORVCC的使能。
     
     在各个运行模式下，NORVCC的使能状态与NORM_NORVCC_EN和STBY_NORVCC_EN的关系如下：

     .. image:: ../../_static/norvcc_en.png
      :align: center
      
     当NORVCC处于OFF状态时，NORVCC的引脚可用作GPIO19。

低功耗场景下的Nor Flash接入策略要点
---------------------------------------

当 Nor Flash的VDD引脚有电时， NSS引脚如果处于浮空状态，则会造成300uA左右的漏电，因此在低功耗状态下，Nor Flash的策略要点是：
 1. 当Nor Flash的VDD引脚有电时，需要处置NSS引脚为高电平（通过IO驱动NSS为高电平或者使用上拉的方式维持NSS为高电平）。
 2.	当Nor Flash的VDD引脚无供电时，需要处置NSS引脚为悬空状态，避免NSS引脚上的漏电。

低功耗场景下的Nor Flash接入策略详情
---------------------------------------

SPV1x的Nor Flash供电有3种方式，每种方式需要搭配对应的软件策略，以尽可能降低Nor Flash在低功耗状态下的电流消耗。

 1. 使用NORVCC（GPIO19）供电

  .. image:: ../../_static/norflash-vdd-connect1.png
    :align: center

  这是推荐的Nor Flash供电方式。当芯片进入休眠模式时，NORVCC的供电会自动切断，Nor Flash处于完全掉电状态，这可以进一步降低整体的电流消耗。
 
  芯片软件配置与各种运行模式下的Nor Flash供电情况如下：

  .. image:: ../../_static/norflash-vdd-cfg1.png
    :align: center

 2. 使用IOVCC供电

   .. image:: ../../_static/norflash-vdd-connect2.png
    :align: center

  使用这种方式供电，可以释放NORVCC引脚为GPIO19功能。由于IOVCC无法彻底断电，因此需要保证芯片在各种运行模式下，NSS引脚的空闲电平都为高电平。典型的情况是芯片进入休眠模式时，负责连接NSS的GPIO会掉电，从而无法输出高电平，这个时候需要设置NORM_NORNSS_PU=1，开启额外的上拉电阻，以保证NSS为高电平。
 
  芯片软件配置与各种运行模式下的Nor Flash供电情况如下：

  .. image:: ../../_static/norflash-vdd-cfg2.png
    :align: center

 3. 使用外部供电

   .. image:: ../../_static/norflash-vdd-connect3.png
    :align: center

  这种方式几乎与“方式2”一致，只是旁路了内部的LDO_IOVCC。在软件配置上与“方式2”也一致。

  芯片软件配置与各种运行模式下的Nor Flash供电情况如下：

  .. image:: ../../_static/norflash-vdd-cfg3.png
    :align: center

.. note::

 注意事项：
  IOVCC引脚上的电压由LDO_SIOVCC和LDO_IOVCC共同提供（取决于输出电压较大的那个）。LDO_IOVCC在ACTIVE和SLEEP模式下处于开启状态；在STANBY和HIBERNATE模式下处于关闭状态。LDO_SIOVCC在所有运行模式下都处于开启状态。由于LDO_SIOVCC只有5mA的驱动能力，因此在芯片进入STANBY和HIBERNATE模式时，需要关闭IOVCC引脚上较为耗电的负载，以避免LDO_SIOVCC过载。

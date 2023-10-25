.. _key-module:

按键 (Keys)
======================

简介
-------------------------

SPV1x的按键API提供IO按键，AD按键和MATRIX按键（矩阵按键）功能。

通过在 `tick_handler_10ms()` 中调用 `key_scan()` ，按键API将以10ms的间隔扫描按键，并将符合要求的按键事件发送到消息队列。

按键事件对应的消息中，使用2个参数来表示具体的事件信息： `id` 字段表示哪一个按键， `val` 字段表示该按键的状态。按键状态在  `key.h` 中以枚举的形式提供，后续的内容将描述这些状态的产生时机。

.. c:enum:: KEY_STA_Type

  按键的状态枚举定义。

   - *KEY_STA_NONE*：未使用。
   - *KEY_STA_SHORT*：按键短按。
   - *KEY_STA_SHORT_UP*：按键短按抬起。
   - *KEY_STA_LONG*：按键长按。
   - *KEY_STA_HOLD*：按键保持。
   - *KEY_STA_LONG_UP*：按键长按抬起。
   - *KEY_STA_DOUBLE*：按键双击。


按键API的全局配置
-------------------------

在 `key.h` 中，有对按键API的全局配置。主要配置项如下：

 1. 按键数量配置

 .. code-block:: c

   #define	KEY_NUM			(16)	//按键数量,0~255

 系统支持的最大按键数量。根据实际需求进行配置，保证数量够用即可，数量越多，RAM和处理逻辑耗时越多。

 2.	按键双击功能配置

  (1) 双击功能使能配置

  .. code-block:: c

    #define KEY_DOUBLE_CLICK_EN		1	//按键双击功能使能

  双击功能会消耗额外的RAM，如果实际不会用到双击功能，请设置为关闭。

  (2) 双击间隔时间配置

  .. code-block:: c

   #define KEY_DOUBLE_CLICK_CNT    35 //35*10ms

  双击间隔。在第一次按键短按并松开后，如果第二次按键在 `KEY_DOUBLE_CLICK_CNT` 设定的时间内短按并松开，就会产生按键的双击事件。

 3. 按键时间阈值配置

  (1) 按键消抖时间配置

  .. code-block:: c

   #define KEY_BASE_CNT	(2)		//按键基础滤波，2*10ms

  `KEY_BASE_CNT` 用于设定按键消抖时间。

  (2) 按键短按时间配置

  .. code-block:: c

   #define KEY_SHORT_CNT	(3)		//短按生效时间，3*10ms

  `KEY_SHORT_CNT` 用于设定按键短按生效时间。当按键的按下时间达到 `KEY_SHORT_CNT` 设定的时间时，将产生按键短按事件 `KEY_STA_SHORT` 。

  (3) 按键长按时间配置

  .. code-block:: c

    #define KEY_LONG_CNT	(75)	//长按生效时间，75*10ms

  `KEY_LONG_CNT` 用于设定按键长按生效时间。当按键的按下时间达到 `KEY_LONG_CNT` 设定的时间时，将产生按键长按事件 `KEY_STA_LONG` 。

  (4) 按键保持时间配置

  .. code-block:: c

   #define KEY_HOLD_CNT	(15)	//保持生效时间，15*10ms

  `KEY_HOLD_CNT` 用于设定按键保持生效时间。当按键的按下时间达到 `KEY_LONG_CNT+ KEY_HOLD_CNT` 设定的时间后，按键保持就会激活，并发送按键保持事件，此后，按键API将以 `KEY_HOLD_CNT` 设定的时间周期性发送按键保持事件。按键保持事件可用实现按键的连发（机打）功能。

 .. note::

  当按键松开时，将会产生按键抬起事件，其具体情形如下：

  1. 当按键抬起前，其按下时间小于 `KEY_SHORT_CNT` 设定的时间时，不会产生抬起事件。

  2. 当按键抬起前，其按下时间大于等于 `KEY_SHORT_CNT` 设定的时间，但小于 `KEY_LONG_CNT` 设定的时间时，将产生短按抬起事件 `KEY_STA_SHORT_UP` 。

  3. 当按键抬起前，其按下时间大于等于 `KEY_LONG_CNT` 设定的时间，将产生长按抬起事件 `KEY_STA_LONG_UP` 。

 4. 按键类型使能配置

 .. code-block:: c

  #define KEY_IO_EN             1   ///<IO按键使能
  #define KEY_AD_EN             1   ///<AD按键使能
  #define KEY_MATRIX_EN         0   ///<矩阵按键使能
  #define KEY_IR_EN             0   ///<IR按键使能
  #define KEY_TOUCH_EN          0   ///<触摸按键使能

.. note::

 目前IR按键和TOUCH按键还未实现。


IO按键
-------------------------

如果使能IO按键，则需要在 `key_io.c` 中对其进行配置：

.. code-block:: c

 //IO按键引脚列表
 static const uint8_t iokey_pins[] =
 {
 	GPIO_Pin_24,	//1
 	GPIO_Pin_25,	//2
 	GPIO_Pin_14,	//3
 	GPIO_Pin_22,	//4
 	GPIO_Pin_18,	//5
 	GPIO_Pin_05,	//6
 	GPIO_Pin_16,	//7
 	GPIO_Pin_09,	//8
 	GPIO_Pin_17,	//9
 	GPIO_Pin_15,	//10
 	GPIO_Pin_26,	//push
 };
 
 //IO按键引脚和键值映射表
 static const uint8_t pins2bitmap[] =
 {
 	KEY_VAL_0,
 	KEY_VAL_1,
 	KEY_VAL_2,
 	KEY_VAL_3,
 	KEY_VAL_4,
 	KEY_VAL_5,
 	KEY_VAL_6,
 	KEY_VAL_7,
 	KEY_VAL_8,
 	KEY_VAL_9,
 	KEY_VAL_10
 };

用户只需要在 `iokey_pins[]` 中配置用到的引脚，在 `pins2bitmap[]` 中配置对应的事件ID即可。

当 `iokey_pins[]` 中的对应引脚检测到按键事件时，按键API会查找 `pins2bitmap[]` 中的数据，并作为事件ID进行发送。

AD按键
-------------------------

如果使能AD按键，则需要在 `key_ad.c` 中对其进行配置：

 1. 配置AD按键用到的引脚

 .. code-block:: c

  static const uint8_t adkey_pins = GPIO_Pin_00;

 可以作为AD按键使用的引脚如下：
 
 (1) GPADC_Signal_GPIO00
 (2) GPADC_Signal_GPIO01
 (3) GPADC_Signal_GPIO02
 (4) GPADC_Signal_GPIO03
 (5) GPADC_Signal_GPIO04
 (6) GPADC_Signal_GPIO24
 (7) GPADC_Signal_GPIO25
 (8) GPADC_Signal_GPIO26

 2. 配置AD按键的数量和电阻值

 .. code-block:: c

   #define KEY_AD_NUM       (3)
   #define KEY_AD_RES_B     (22.0f)
   #define KEY_AD_RES_U1    (6.8f)
   #define KEY_AD_RES_U2    (3.3f)
   #define KEY_AD_RES_U3    (0.0f)

 配置 `KEY_AD_NUM` 为实际应用中的AD按键数量。

 配置 `KEY_AD_RES_B` 为高侧分压电阻的阻值（单位KΩ）。

 配置 `KEY_AD_RES_U1` 为1号按键的低侧分压电阻的阻值（单位KΩ）。同理，依次配置 `KEY_AD_RES_U2` 、 `KEY_AD_RES_U3` ……

 3. 配置AD按键分压表

 .. code-block:: c

  const uint16_t ad_key_value_table[][KEY_AD_NUM] =
  {
    {
        KEY_AD_REG_VAL(KEY_IOVCC_2V0,KEY_AD_RES_U1,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V0,KEY_AD_RES_U2,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V0,KEY_AD_RES_U3,KEY_AD_RES_B)
    },
    {
        KEY_AD_REG_VAL(KEY_IOVCC_2V2,KEY_AD_RES_U1,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V2,KEY_AD_RES_U2,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V2,KEY_AD_RES_U3,KEY_AD_RES_B)
    },
    {
        KEY_AD_REG_VAL(KEY_IOVCC_2V4,KEY_AD_RES_U1,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V4,KEY_AD_RES_U2,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V4,KEY_AD_RES_U3,KEY_AD_RES_B)
    },
    {
        KEY_AD_REG_VAL(KEY_IOVCC_2V6,KEY_AD_RES_U1,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V6,KEY_AD_RES_U2,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V6,KEY_AD_RES_U3,KEY_AD_RES_B)
    },
    {
        KEY_AD_REG_VAL(KEY_IOVCC_2V8,KEY_AD_RES_U1,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V8,KEY_AD_RES_U2,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_2V8,KEY_AD_RES_U3,KEY_AD_RES_B)
    },
    {
        KEY_AD_REG_VAL(KEY_IOVCC_3V0,KEY_AD_RES_U1,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_3V0,KEY_AD_RES_U2,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_3V0,KEY_AD_RES_U3,KEY_AD_RES_B)
    }, 
    {
        KEY_AD_REG_VAL(KEY_IOVCC_3V2,KEY_AD_RES_U1,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_3V2,KEY_AD_RES_U2,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_3V2,KEY_AD_RES_U3,KEY_AD_RES_B)
    },
    {
        KEY_AD_REG_VAL(KEY_IOVCC_3V4,KEY_AD_RES_U1,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_3V4,KEY_AD_RES_U2,KEY_AD_RES_B),
        KEY_AD_REG_VAL(KEY_IOVCC_3V4,KEY_AD_RES_U3,KEY_AD_RES_B)
    },
  };

 由于芯片的IOVCC会随着芯片VCC供电的变化而变化，因此需要配置在不同IOVCC下，各个按键分压后的ADC转换值。

 4. 配置AD按键ID映射表

 .. code-block:: c

  static const uint8_t adkey_pins2bitmap[] =
  {
    KEY_VAL_11,
    KEY_VAL_12,
    KEY_VAL_13,
  };

 当AD按键按下时，按键API会从 `adkey_pins2bitmap[]` 获取对应按键的事件ID。

 5. 配置AD按键的ADC值裕度

 .. code-block:: c

  #define ADKEY_MARGIN      (200)

 ADC值的裕度需要根据实际项目来调整。
 假定按键A按下后，ADC的理论值为 `val` ，那么实际的ADC值只要满足大于 `(val - ADKEY_MARGIN)` 且小于 `(val + ADKEY_MARGIN)` ，就会认为按键A按下。

 .. note::

  1. ADC的量程为3V，分辨率为12bit。测量超过3V的电压时，其结果始终为0xfff。

  2. 不同按键分压后的电压值应尽量分散，以充分利用ADC的量程，增加抗干扰能力。

  3. AD按键不适合做多键同时检测。

矩阵按键
-------------------------

如果使能矩阵按键，则需要在 `key_matrix.c` 中对其进行配置：

.. code-block:: c

 static const uint8_t key_matrix_seg_pins[] =
 {
 	GPIO_Pin_00,	//SEG0
 	GPIO_Pin_01,	//SEG1
 	GPIO_Pin_05,	//SEG2
 };
 
 static const uint8_t key_matrix_com_pins[] =
 {
 	GPIO_Pin_07,	//COM0
 	GPIO_Pin_14,	//COM1
 	GPIO_Pin_15,	//COM2
 	GPIO_Pin_21,	//COM3
 };
 static const uint8_t pins2bitmap[] =
 {
 	KEY_VAL_0,
 	KEY_VAL_1,
 	KEY_VAL_2,
 	KEY_VAL_3,
 	KEY_VAL_4,
 	KEY_VAL_5,
 	KEY_VAL_6,
 	KEY_VAL_7,
 	KEY_VAL_8,
 	KEY_VAL_9,
 	KEY_VAL_10,
 	KEY_VAL_11,
 };


在 `key_matrix_seg_pins[]` 中配置矩阵按键的SEG口。在 `key_matrix_com_pins[]` 中配置矩阵按键的COM口。在 `pins2bitmap[]` 中配置对应按键的事件ID。

以上面的配置为例， `pins2bitmap[]` 中，事件ID与按键对应关系为：

1. `pins2bitmap[0]` 对应 `COM0.SEG0` ;
2. `pins2bitmap[1]` 对应 `COM0.SEG1` ;
3. `pins2bitmap[2]` 对应 `COM0.SEG2` ;
4. `pins2bitmap[3]` 对应 `COM1.SEG0` ;
5. `pins2bitmap[4]` 对应 `COM1.SEG1` ;
6. `pins2bitmap[5]` 对应 `COM1.SEG2` ;
7. `pins2bitmap[6]` 对应 `COM2.SEG0` ;
8. `pins2bitmap[7]` 对应 `COM2.SEG1` ;
9. `pins2bitmap[8]` 对应 `COM2.SEG2` ;
10. `pins2bitmap[9]` 对应 `COM3.SEG0` ;
11. `pins2bitmap[10]` 对应 `COM3.SEG1` ;
12. `pins2bitmap[11]` 对应 `COM3.SEG2` ;


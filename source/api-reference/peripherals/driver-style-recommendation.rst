外设驱动编程建议
======================

外设寄存器的访问方式
----------------------

SPV1x SDK的外设头文件提供两种方式进行Memory-mapped外设寄存器的访问：

 1. 通过 DOMAIN_PERIPHERAL->REGISTER 的格式，读写电源域 DOMAIN 下外设名 PERIPHERAL 的 32bit位宽寄存器 REGISTER。
    
    例如，使用 DEV_GPIO->OUT0 访问主电源域下GPIO外设的输出寄存器OUT0。
    
    其中，电源域 DOMAIN 名称为 “DEV” (主电源域) 和 “AON” (Always-On电源域) 二者之一，由外设实际所在的电源域所决定。

    这种访问方式非常适合对外设控制寄存器进行复杂的整体配置，例如对PWM通道0的控制寄存器CTL进行初始化：

    .. code-block:: 

      DEV_PWM->CTL[0] =
			DEV_PWM_CTL_SoftMode_OUT_SET	        |
			DEV_PWM_CTL_SoftMode_MOD_OUT	        |
			DEV_PWM_CTL_SoftMode_OUT_FMT_OVERTURN	|
			(4ul << 16)                             |
			DEV_PWM_CTL_SoftMode_FMT_2DATA	        |	
			(10ul << 4)                             |
			DEV_PWM_CTL_SoftMode_PWM_OE             |	
			DEV_PWM_CTL_SoftMode_MODE_SOFTMODE      |	
			DEV_PWM_CTL_SoftMode_EN;

    .. note::
   
     后续外设说明中的“寄存器定义”章节，将提供每个外设寄存器的 **API 名称** ，即用户程序中调用指定外设所需的代码段，请留意。		         

 2. 通过 DOMAIN_PERIPHERAL->REGISTER_f.BITFIELD 的格式，访问电源域 DOMAIN 下外设 PERIPHERAL的 REGISTER 寄存器
    的指定位域 BITFIELD。
    
    例如，通过 DEV_GPIO[0]->CTL_f.PU=1 开启GPIO0控制寄存器CTL的上拉使能位(PU)。

    这种访问方式，适合对寄存器的特定位域进行独立的读写操作。
    相比于第一种方式，用户不必考虑连带性的位域遮蔽/清除等操作，编译器将负责实际代码的生成。
    但是，通过此种方式连续针对相同寄存器的不同位域进行操作，相比于方式一，将生成可观的冗余代码，无助于代码性能的保持。

.. warning::
    - 如果使用struct + bitfield的方式访问外设寄存器，请 **务必** 将 *-fstrict-volatile-bitfields* 作为compiler flag (CFLAG)传递于编译器。

外设初始化方法
----------------------

外设的初始化可以划分成以下典型步骤：

 1. 根据场景要求或限制，选择该外设的驱动时钟源。如果该驱动时钟源处于关闭状态，用户程序应首先激活该时钟源信号，有必要时需等待该时钟源信号稳定。激活特定时钟源的操作可能涉及 ``电源管理单元(PMU)`` , ``振荡器单元(OSC)`` 等模块的合理配置。
 2. 通过 ``时钟管理单元(CMU)`` 配置外设的驱动时钟源及可选配的预分频系数，然后通过CMU_CLKEN寄存器开启该外设的驱动时钟通路。
 3. 通过 ``复位管理单元(RMU)`` RMU_RSTEN寄存器将该外设从复位状态释放。
 4. 配置该外设自身寄存器组。如果以中断响应方式使用该外设，需要预先配置中断控制器(ECLIC)对应通路，并挂载匹配的中断处理函数。如果涉及与DMA单元的联动，还需要妥善配置DMA相关寄存器和中断响应。
 5. 对于使用GPIO进行数据传输的外设(如UART、ADC)，还需要通过 ``GPIO单元`` 的多功能复用(MFP)，分配合适的GPIO管脚。

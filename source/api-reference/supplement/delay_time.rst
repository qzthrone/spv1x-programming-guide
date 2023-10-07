.. _delay-api-ref:

延时
======================

.. c:function:: void delay_us(uint32_t us)

  执行指定微秒数的阻塞式延时。

  :param us: 延时时间，单位为微秒。在CPU最高主频120MHz情况下，可以提供最多35790微秒的延时配置。
  :returns: 无

.. c:function:: void delay_ms(uint32_t ms)

  执行指定毫秒数的阻塞式延时。

  :param us: 延时时间，单位为毫秒。在CPU最高主频120MHz情况下，可以提供最多35790毫秒的延时配置。
  :returns: 无






   



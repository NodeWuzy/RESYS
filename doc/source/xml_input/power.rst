==============================
 反应堆堆芯功率与点堆动力学
==============================

在RESYS程序中，点堆动力学在输入文件中使用标签 ``<pointReactor>`` 定义。并且在程序中点堆动力学采用Gear算法求解，兼具有高精度以及高计算速度，可以自动变阶变步长。
下面是一个点堆动力学输入的例子。

.. code-block:: xml

    <!--****************************************************************-->
    <!--*                         点堆模型                             *-->
    <!--****************************************************************-->
    <pointReactor
        name                                   =     "reactor_power"
        lambda                                 =     "0.0127     0.0317     0.1150     0.3110     1.4000     3.8700"
        beta                                   =     "0.266E-3   1.491E-3   1.316E-3   2.849E-3   0.869E-3   0.182E-3"
        Tao                                    =     "2.0E-5"
        max_time_step                          =     "0.1"
        min_time_step                          =     "1.0E-12"
        gear_max_order                         =     "6"
        reactivity                             =     "reactivity_time_function"
        enable_reactivity_feed_back            =     "true"
        reactivity_linearization               =     "true"
        start_feed_back_time                   =     "10500"
        feed_back_time_range                   =     "10500 11080"
    />

- ``lambda``                      :  缓发中子先驱核的衰变常数。
- ``beta``                        ： 缓发中子先驱核的份额。
- ``Tao``                         ： 中子的平均寿期。
- ``max_time_step``               ： 最大时间步长。
- ``min_time_step``               :  最小的时间步长。
- ``gear_max_order``              ： gear算法变阶时最大的阶数，不超过6。
- ``reactivity``                  ： 外部输入的反应性系数。
- ``enable_reactivity_feed_back`` :  是否启用反应性反馈。
- ``reactivity_linearization``    ： 是否线性化反馈反应性的变化。
- ``start_feed_back_time``        ： 开始反应性反馈计算的时间，默认为0 。
- ``feed_back_time_range``        ： 计算反应性反馈的时间区间，超过这个区间则不计算反应性反馈。本项可以不输入。

反应堆堆芯功率使用 ``<reactorPower>`` 标签来输入。

.. code-block:: xml

    <!--****************************************************************-->
    <!--*                        反应堆堆芯功率                         *-->
    <!--****************************************************************-->
    <reactorPower
        initial_power                        =     "1.0"
        use_pointReactor_model               =     "false"
        iteration_max_times                  =     "100"
        effect_by_pointRector                =     "false"
        time_dependent_factor                =     "power_time_func"
    />

- ``initial_power``             :   初始的功率因子。
- ``use_pointReactor_model``    ：  是否使用点堆动力学。
- ``iteration_max_times``       ：  最大的迭代次数，通常指的反应性反馈迭代。
- ``effect_by_pointRector``     ：  反应堆功率是否被点堆动力学模型控制。
- ``time_dependent_factor``     ：  反应堆功率随着时间的变化因子。
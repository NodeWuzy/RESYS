===================================
热构件及其相关模型
===================================

带有热构件的管型部件(pipeWithHeatStructure)
=======================================================

``pipeWithHeatStructure`` 是一个带有热构件的管型部件，其表述这样的物理模型：冷却剂从管型部件中流过，与热构件之间发生对流换热。
其物理过程如下图中所示：

.. image:: /_static/pipewithHS.png
   :height: 398px
   :width:  500px
   :align: center

下面是一个使用pipeWithHeatStructure模拟管式辐射散热器及其翅片的例子。

.. code-block:: xml

    <!--****************************************************************-->
    <!--*    使用pipeWithHeatStructure模拟管式辐射散热器及其翅片          *-->
    <!--****************************************************************-->   
    
    <pipeWithHeatStructure

        name                                 =     "radiatorPipe3"
        orientation                          =     "0    0     -1" 
        start_position                       =     "0.2  0  0.375"
        n_elems                              =     "40"
        pipe_area                            =     "3.84E-5"
        pipe_length                          =     "1.848"
        friction                             =     "0.3"
        Hw                                   =     "100000.0"
        area_scale_factor                    =     "19.5"
    
        hs_type                              =     "slab"
        hs_name                              =     "copper_fin"
        wall_thick                           =     "0.025"
        wall_depth                           =     "0.4E-3"
        wall_material                        =     "copper"
        wall_elems                           =     "4"
        wall_dim                             =     "2"
        wall_heat_source                     =     "radiator_heat_source"
        heat_flux_scale_factor               =     "39.0"        

    />

上面的输入中前面一部分见 管型部件 ``<pipe>`` 的输入，其与之相同。后面一部分的输入为热构件的输入。

- ``hs_type``           : 热构件的类型，可以是板状热构件 ``slab`` 、 ``cartesian`` 、 ``squre`` 或者 圆柱状的热构件 ``cylinder`` 、 ``cylinrical`` 。
- ``hs_name``           ：热构件的名称。
- ``wall_thick``        : 热构件的宽度。
- ``wall_depth``        ：热构件的厚度，只有方形的热构件才输入。
- ``wall_material``     : 热构件的材料。
- ``wall_dim``          ：根据是否计算轴向热传导确定热构件的维度，如果只计算横向热传导则为1，否则为2。
- ``wall_elems``        ：热构件横向的网格划分数。
- ``wall_heat_source``  ：壁面与冷却剂流道的界面的热流密度缩放因子。

单通道反应堆堆芯模型
============================================================

单通道堆芯模型与带有热构件的管型部件模型类似，在程序中均是由一个管型和一个热构件组成。

.. image:: /_static/corechannel.png
   :height: 330px
   :width:  400px
   :align: center

下面是一个单通道堆芯模型的输入。

.. code-block:: xml
   
   <coreChannel
    
        name                       =     "coreChannel"
        orientation                =     "1 0 0"
        start_position             =     "0 0 0"
        n_elems                    =     "20"
        pipe_area                  =     "2.813E-4"
        pipe_length                =     "60.80E-2"
        pipe_Dh                    =     "0.216E-2"
        Hw                         =     "Hissam"
        friction                   =     "Blasius"
        
        geometry_type              =     "cylinder"
        
        hs_layer_number            =     "3"
        axial_power_shape          =     "axial_power_shape_func"
        hs_dimension               =     "1"
        hs_matrial                 =     "UO2 gap_mat Stainless_Steel"
        hs_thick                   =     "0.9095E-2 0.022E-2 0.051E-2"
        hs_elems_number            =     "10 1 1"
        power_fraction             =     "1 0 0"
        hs_init_T                  =     "800"
        power                      =     "3.47916E3"
   
   />


与前面一样，上半部分的输入是关于管型部件 ``pipe`` 的参数，后面的参数是关于热构件的参数。

- ``geometry_type``      :  燃料元件的几何类型，为圆柱形 ``cylinder`` 或者板型 ``slab`` 。
- ``hs_layer_number``    ： 燃料元件或者热构件的层数。
- ``axial_power_shape``  :  燃料沿着轴向的功率分布函数。
- ``hs_matrial``         ： 热构件的材料。
- ``hs_thick``           ： 燃料(热构件)每层的厚度。
- ``hs_elems_number``    ： 燃料(热构件)每层的分层数。
- ``power_fraction``     ： 燃料(热构件)每层的功率分布。
- ``hs_init_T``          ： 燃料(热构件)每层的初始温度。
- ``power``              ： 燃料(热构件)的功率水平。


换热器模型
=============================================================

换热器是由一个由两个管型部件 ``pipe`` 和一个热构件组成。
如下图中所示，冷却剂从两个管型部件中流过通过热构件之间发生热量交换。
换热器一次侧与二次侧的冷却剂只发生热量传递，但是一、二次侧的流动之间彼此隔绝，不发生质量与动量交换。

换热器的模型的示意如下所示：

.. image:: /_static/heatExchanger.png
   :height: 648px
   :width:  500px
   :align: center



   






































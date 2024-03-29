-------------------------
热离子燃料元件与TFE系统
-------------------------


热离子燃料元件
----------------

热离子燃料元件是一个非常复杂的部件。
在程序中，其由多个热构件以及一个冷却剂流道(管型部件组成)。
这个部件设计的目的是为了简化在对热离子反应堆进行模拟时的输入，提供模拟热离子发射静态热电转换的接口。

一个堆内热离子燃料元件模型由如下部分构成：

- 核燃料热构件
- 发射极热构件
- 接受极与绝缘极热构件
- 内不锈钢套筒热构件
- 冷却剂流道管型部件 
- 外不锈钢套筒热构件
- 燃料--发射极耦合界面
- 发射极--接受极热离子耦合界面
- 绝缘体--内不锈钢套筒耦合界面
- 内不锈钢套筒--冷却剂流道对流耦合界面
- 外不锈钢套筒--冷却剂流道对流耦合界面

热离子燃料元件TFE的横截面图如下中所示：

.. image:: /_static/thermionicfuelcell.png
   :height: 330px
   :width:  400px
   :align: center

因此要模拟热离子燃料元件时，需要在输入文件中定义上面列表中给出的部件。
这会使得输入文件非常冗长并且难以维护。
因此，在程序中开发了一个专门的热离子燃料元件模型以简化在模拟热离子堆时的输入。

热离子燃料元件在xml输入文件中使用标签 ``<thermionicFuelCell>`` 来表示，
下面给出了在输入文件中一个典型的热离子燃料元件的输入：

.. code-block:: xml

   <thermionicFuelCell
         
   name                                  =     "third_ring_fuel_cell"

   orientation                           =     "0  0  1"
   start_position                        =     "0  0  0"

   central_void_radius                   =     "2.25"
   fuel_outter_radius                    =     "8.5"
   emitter_inner_radius                  =     "8.65"
   emitter_thick                         =     "1.15"
   collector_thick                       =     "1.40"
   collector_inner_radius                =     "10.30"
   insulator_thick                       =     "0.15"
   inner_cladding_inner_radius           =     "11.90"
   inner_cladding_thick                  =     "0.35"
   outer_cladding_inner_radius           =     "12.95"
   outer_cladding_thick                  =     "0.35"

   scale_factor                          =     "0.001"

   axial_cell_length                     =     "11.0[5] 12.50[3] 12.50[24] 12.50[3] 11.0[5]"
   fuel_elems                            =     "5"

   powershape                            =     "axial_power_shape_func"
   fuel_power                            =     "2890.4"

   fuel_material                         =     "UO2"
   collector_material                    =     "collector_mat"
   emitter_material                      =     "emitter_mat"
   insulator_matrial                     =     "Al2O3"
   cladding_matrial                      =     "Stainless_Steel"

   h_fuel_gap                            =     "4000"
   electrode_gap_gas                     =     "Mixtrue_gas"
   insulator_gap_gas                     =     "He_gas"

   flow_path_friction                    =     "0.1"
   flow_path_Hw                          =     "Dwyer_Tu"
   only_radial_heat_transfer             =     "false"

   fuel_reactivity                       =     "fuel_feed_back_func"
   emitter_reactivity                    =     "electrodes_feed_back_func"
   collector_reactivity                  =     "electrodes_feed_back_func"
   fuel_reactivity_weight                =     "0.00486486"
   emitter_reactivity_weight             =     "0.00486486"
   collector_reactivity_weight           =     "0.00486486"

   heat_flux_scale_factor                =     "18"
   flow_path_scale_factor                =     "18"

   has_axial_reflector                   =     "true"
   reflector_material                    =     "BeO"
   upper_reflector_cell_num              =     "5"
   lower_reflector_cell_num              =     "5"

   CS_pressure                           =     "Cs_press_time_func"	  
   active_region_index                   =     "5   34"

   />

下面给出热离子燃料元件 ``<thermionicFuelCell>`` 标签中的每一个属性的意义：

- ``name``                        ： 热离子燃料元件的名称，也是热离子燃料元件中管型部件的名称。
- ``orientation``                 ： 热离子燃料元件中管型部件的朝向。
- ``start_position``              ： 热离子燃料元件中管型部件的起始位置。
- ``central_void_radius``         ： 热离子燃料元件中心孔半径。
- ``fuel_outter_radius``          ： 热离子燃料元件燃料外半径。
- ``emitter_inner_radius``        ： 热离子燃料元件发射极内半径。
- ``emitter_thick``               ： 热离子燃料元件发射极厚度。
- ``collector_thick``             ： 热离子燃料元件接受极厚度。
- ``collector_inner_radius``      ： 热离子燃料元件接受极内半径。
- ``insulator_thick``             :  热离子燃料元件绝缘体厚度
- ``inner_cladding_inner_radius`` ： 热离子燃料元件内套管内半径。
- ``inner_cladding_thick``        ： 热离子燃料元件内套管厚度。
- ``outer_cladding_inner_radius`` ： 热离子燃料元件外套管内半径。
- ``outer_cladding_thick``        ： 热离子燃料元件外套管厚度。
- ``length``                      ： 热离子燃料元件的长度。
- ``scale_factor``                ： 热离子燃料元件的长度尺度单位缩放。
- ``axial_elems``                 ： 热离子燃料元件轴向分段数。
- ``fuel_elems``                  ： 热离子燃料元件的燃料热构件分段数。

- ``powershape``                  :  热离子燃料元件的轴向功率分布函数，输入常数或者函数名称。
- ``power``                       :  热离子燃料元件热功率，单位W。
- ``fuel_material``               :  燃料的材料。
- ``collector_material``          ： 接受极的材料，输入材料名称。
- ``emitter_material``            ： 发射极的材料，输入材料名称。
- ``insulator_matrial``           ： 绝缘极的材料，输出材料名称。
- ``cladding_matrial``            ： 包壳的材料  ，输入材料名称。
- ``h_fuel_gap``                  ： 燃料-包壳间隙的等效换热系数。
- ``electrode_gap_gas``           ： 接收极-发射极间隙的填充间隙名称。
- ``h_electrode_gap``             ： 接收极-发射极间隙等效换热系数。
- ``insulator_gap_gas``           ： 绝缘极-内套管间隙填充气体的名称。
- ``h_insulator_gap``             ： 绝缘极-内套管间隙等效换热系数。
- ``flow_path_friction``          ： 冷却剂流道流动阻力。
- ``flow_path_Hw``                ： 冷却剂流道对流换热系数。
- ``flow_path_init_v``            :  冷却剂流道初始流动速度，见管型部件输入 ``init_v`` 。
- ``flow_path_init_p``            ： 冷却剂流道初始压力分布，见管型部件输入 ``init_p`` 。
- ``flow_path_init_T``            ： 冷却剂流道初始温度分布，见管型部件输入 ``init_T`` 。
- ``only_radial_heat_transfer``   :  热离子燃料元件是否只计算轴向传热。
- ``fuel_reactivity``             ： 燃料的反应性反馈函数
- ``emitter_reactivity``          :  发射极的反应性反馈函数
- ``collector_reactivity``        :  接收极的反应性反馈函数
- ``fuel_reactivity_weight``      :  燃料的反应堆反馈权重
- ``emitter_reactivity_weight``   :  发射极的反应性反馈权重
- ``collector_reactivity_weight`` :  接受极的反应性反馈权重
- ``heat_flux_scale_factor``      :  由热离子燃料元件传入到冷却剂流道的热流密度的缩放因子，通常为模型代表的同类型的热离子燃料元件的数量。
- ``lower_reflector_cell_num``    :  热离子燃料元件冷却剂流动面积的缩放因子，通常为模型代表的同类型的热离子燃料元件的数量。
- ``has_axial_reflector``         ： 是否拥有轴向反射层。
- ``reflector_material``          ： 反射层的材料
- ``upper_reflector_cell_num``    :  上反射层的控制体数。
- ``lower_reflector_cell_num``    ： 下反射层的控制体数。
- ``CS_pressure``                 :  铯压随着时间的变化，输入常数或者时间的常数。
- ``active_region_index``         ： 接受极和发射极能够发射热电子的有效区域的控制体编号。

.. note::

   对于某一间隙的等效换热系数与间隙的填充气体类型只能输入其中一种，而不能都输入。


TFE系统电气系统
-------------------------

.. code-block:: xml

   <auxilary_model
      model_type                            =     "TFELoadSys"
      thermionic_fuel_cell_name             =     "center_fuel_cell           first_ring_fuel_cell  
                                                   second_ring_fuel_cell      third_ring_fuel_cell"
      thermionic_fuel_cell_num              =     "1       6        12       15"
      closed_time                           =     "1500"
      Re2                                   =     "0.97E-3"
      Rc2                                   =     "0.21E-3"
      Re1                                   =     "0.64E-3"
      Rc1                                   =     "0.38E-3"      
      RL                                    =     "0.14800"
    />


在RESYS程序中，TFE电气系统是一个辅助模型，因此使用 ``auxilary_model`` 来指定，并且 ``model_type`` 设置为 ``TFELoadSys``。
并且在本模型中，单节型热离子燃料元件使用双端串联的方式给负载系统提供电源。


下面给出输入的每一个变量的定义：

- ``thermionic_fuel_cell_name``             :   给负载系统提供电源的热离子燃料元件的名称。
- ``thermionic_fuel_cell_num``              ：  给负载系统提供电源的热离子燃料元件的数量。
- ``closed_time``                           ：  负载系统开始运行的时间，如果不输入，默认为0。
- ``Re2`` 、 ``Re1``                        ：  发射极的接线电阻的大小。
- ``Rc2`` 、 ``Rc1``                        :   接受极的接线电阻的大小。
- ``RL``                                    :   负载电阻的电阻大小。





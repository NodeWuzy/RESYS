=======================================
流场与插值
=======================================

在求解流体动力学方程组的时候，需要用到流场的插值。
比如密度的信息是存储在单一控制体中心的。
在求解动量方程的时候，需要用到单一控制体界面上的密度值，这时就需要从网格中心向网格界面上插值。

另外一个例子是在求解动量方程时，动量方程的对流项使用单一控制体网格中心的速度表示。
但是在求解时，流体的速度却是在网格的界面上得到的，这个时候需要将流体速度信息从界面向网格中心插值。


网格中心向界面插值
=====================

- 线性插值

网格界面上的物理场的值可以使用网格两端控制体的物理场的值线性插值获得：

.. math::

    \phi_f = g_C \phi_C + ( 1 - g_C ) \phi_F

对于线性插值，加权系数 :math:`g_C` 与控制体的几何有关。

.. math::

    g_C =  \frac{ d_{Ff} } { d_{Cf} + d_{Ff}}   

- 调和平均插值

调和平均插值用于计算界面上的扩散系数：

.. math::

    \phi_f = \frac{\phi_C \phi_F}{ g_C \phi_C + ( 1 - g_C )\phi_F }


- 迎风差分插值

迎风差分用于对流项的计算：

.. math::
    \phi_f = ||m_{CF} , 0 || \phi_C - ||- m_{CF}, 0 ||  \phi_F


网格界面物理量向网格中心插值
=================================

这种情况用于利用动量方程中求解出的速度获取网格中心的速度，以及对动量方程对流项使用的迎风差分。
在RESYS程序中，网格界面向网格中心插值的方式和RELAP5中使用的方式类似。

网格中心的速度可以使用与网格中心相连的接管的速度加权平均表示，不失普遍性，可以用公式表示为：

.. math::

   v_{f}^{L}  = \sum_{j=1}^{Jin+Jout+Jside} w_j v_{f,j}

一般情况下，会忽略掉侧面接管对于网格中心的贡献，因此，只需要考虑入口和出口接管对于网格中心平均速度的贡献。这时，上面的公式表示为：

.. math::

   v_{f}^{L}  = \sum_{j=1}^{Jin+Jout} w_j v_{f,j}

在摩擦项的离散中，需要得到网格中心速度的绝对值，最简单的方法是对上面插值得到的值求绝对值。
但是这在两端流动速度相同并且方向相反的时候会出现问题，这会使得得到的网格中心绝对值为0。
在RELAP5中，使用如下的公式解决这个问题：

.. math::
    |v|_{f}^{L}  = \sum_{j=1}^{Jin+Jout} w_j |v_{f,j}|

令人非常高兴的是，对于分支控制体 :class:`volumeBranch` 部件来说。
没有接管连接到其入口和出口，在RESYS程序中所有的接管均连接到分支控制体的侧面。
上面的公式直接给出分支控制体上的流动速度 :math:`v_{f,volumeBranch}^{L} = 0` ，这正是我们想要的结果。


对于单一控制体入口的流速，使用RELAP5手册中给出的公式：

.. math::
    v_{f,in}^{L} = \frac{\sum_{j=1}^{Jin} \dot{\rho}_{f,j} v_{f,j} A_j}{\sum_{j=1}^{Jin} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jin} A_j}{A_L}

当单一控制体入口只有一根接管与之相连时，上面的公式简化为：

.. math::
    v_{f,in}^{L} = \frac{ v_{f,j} A_{f,in}}{A_L}

其中 :math:`v_{f,in}^{L}` 是控制体入口端的流速， :math:`v_{f,j}` 是接管上的流速。 :math:`A_{f,in}` 是接管的面积。
:math:`A_L` 是控制体的流动面积。 

对于单一控制体出口的流速，同样有：

.. math::
    v_{f,out}^{L} = \frac{\sum_{j=1}^{Jout} \dot{\rho}_{f,j} v_{f,j} A_j}{\sum_{j=1}^{Jout} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jout} A_j}{A_L}

上面的公式均为对于单相流动而言的，对于两相流，只需要将上面公式中的 :math:`\dot{\rho}_{f,j}` 替换为 :math:`\dot{\alpha_{f,j}} \dot{\rho}_{f,j}` 。

-----------------
 线性插值：
-----------------

在RELAP5的早期版本中，计算网格中心速度使用的公式为：

.. math::

   v_{f}^{L} = 0.5 (  v_{f,in}^{L} + v_{f,out}^{L}  ) 
   = 0.5 \frac{\sum_{j=1}^{Jin} \dot{\rho}_{f,j} v_{f,j} A_j}{\sum_{j=1}^{Jin} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jin} A_j}{A_L} +
    0.5 \frac{\sum_{j=1}^{Jout} \dot{\rho}_{f,j} v_{f,j} A_j}{\sum_{j=1}^{Jout} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jout} A_j}{A_L}
 
但是之后发现使用0.5作为出口入口速度的权重因子有的时候在动量方程的对流项中会给出非物理的结果。
在随后的版本中，使用了改进的公式：

.. math::

   v_{f}^{L} 
   = \frac{\sum_{j=1}^{Jin} \dot{\rho}_{f,j} v_{f,j} A_j}{\sum_{j=1}^{Jin+Jout} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jin} A_j}{A_L} +
    \frac{\sum_{j=1}^{Jout} \dot{\rho}_{f,j} v_{f,j} A_j}{\sum_{j=1}^{Jin+Jout} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jout} A_j}{A_L}

在RESYS中，使用上面的这个公式来插值计算网格中心的速度。

因此，加权系数定义为：

- ``入口接管权重`` ：

.. math::

    w_j = \frac{ \dot{\rho}_{f,j}A_j}{\sum_{j=1}^{Jin+Jout} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jin} A_j}{A_L}   

- ``出口接管权重`` ：

.. math::

    w_j = \frac{ \dot{\rho}_{f,j} A_j}{\sum_{j=1}^{Jin+Jout} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jout} A_j}{A_L}

- ``侧面接管权重`` ：

.. math::
    w_j = 0

----------------
 迎风差分
----------------
在动量方程的对流项的隐式离散中，使用上面公式进行中心差分会带来不稳定的问题，通常来说，对于对流占优问题，使用迎风差分会具有比较好的稳定性。
在RESYS中，实现了对于动量方程对流项的迎风差分。

首先使用中心插值计算出网格中心的速度 :math:`\bar{v}_{f}^{L}` , 然后利用该速度来进行迎风差分：

.. math::

   v_{f}^{L} = ||\bar{v}_{f}^{L} , 0||  v_{f,in}^{L} - ||- \bar{v}_{f}^{L} , 0||  v_{f,out}^{L}


因此,对于迎风差分格式，其加权系数定义为：

- ``入口接管权重`` ：

.. math::

    w_j = ||\bar{v}_{f}^{L} , 0|| \frac{ \dot{\rho}_{f,j}A_j}{\sum_{j=1}^{Jin} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jin} A_j}{A_L}   

- ``出口接管权重`` ：

.. math::

    w_j = -||- \bar{v}_{f}^{L} , 0|| \frac{ \dot{\rho}_{f,j} A_j}{\sum_{j=1}^{Jout} \dot{\rho}_{f,j} A_j}
    \frac{\sum_{j=1}^{Jout} A_j}{A_L}

- ``侧面接管权重`` ：

.. math::
    w_j = 0

---------------------
RESYS程序的实际实现
---------------------

在RESYS程序中，对于速度的插值的定义在类 ``velocity_interpolation`` 中实现，其定义如下：

    .. code-block:: c++

		//===================================
		// 速度插值父类
		//===================================
		class velocity_interpolation{
			

		public:

			field::junction_field_array      owner_weight;
			field::junction_field_array      neigh_weight;
			
			//-----------------------------------------
			// 构造函数
			//-----------------------------------------
			velocity_interpolation();

			// 从网格界面速度插值得到网格中心速度
			field::volume_field interpolate(
					field::junction_field  density_field  ,
					field::junction_field  velocity_field ,
					field::volume_field    result		  ,
					component::loops       &loop
					) ;

			// 从网格界面速度插值得到网格中心速度的幅度
			// 用于摩擦项的计算
			field::volume_field  interpolate_abs(
					field::junction_field density_field  ,
					field::junction_field velocity_field ,
					field::volume_field   result         ,
					component::loops      &loop
			);

			//-----------------------------------------
			// 虚基函数
			//-----------------------------------------
			// 从网格界面插值计算得到中心速度的权重
			virtual void make_weight(
				field::junction_field density_field , 
				component::loops & loop,
				field::volume_field velocity_center = NULL
		    ) = 0;

		};

类 ``velocity_interpolation`` 是一个虚基类，而实际使用的速度插值函数需要继承该类并且重载其 ``make_weight`` 函数。
对于不同的离散格式，其权重不相同，因此外部调用 ``make_weight`` 函数后，会计算出不同的权重，这些权重保存在类中的变量 ``owner_weight`` 和 ``neigh_weight`` 中。
``owner_weight`` 和 ``neigh_weight`` 是两个 ``junction_field_array`` 类型的变量。
因此它们定义在接管上，``owner_weight`` 保存的是某个接管对于其 ``owner`` 控制体的贡献。
``neigh_weight`` 保存的是某个接管对于其 ``neigh`` 控制体的贡献 。

类 ``velocity_interpolation`` 类提供函数 ``interpolate`` 和 ``interpolate_abs`` 的默认实现，因此子类并不需要重载该函数。
函数 ``interpolate`` 使用计算得到的权重求解计算网格中心的速度插值。
``interpolate_abs`` 计算插值计算网格中心的绝对值，计算得到的该速度绝对值用于动量方程壁面摩擦项的计算。


延时修正
=============

在对流项的隐式离散中，为了提高对流项离散的稳定性，一般而言需要使用迎风差分格式。
但是迎风差分格式往往只具有一阶精度，具有强烈的数值扩散，而高阶迎风格式在隐式离散中不方便使用，往往需要修改稀疏矩阵的格式。
而延时修正方法，虽然会降低迭代的收敛速度，但是其易于应用，是一种比较好的在提高稳定性的情况下获取较高精度计算结果的一种方法。
其构造是使用隐式的迎风差分格式离散，并且方程右端使用显式的高阶格式修正。

延时修正使用的公式如下：

.. math::

    \dot{m_{f}} {\phi_f^{HO}} = [\dot{m_{f}}{\phi_f^{U} }]_{implict} + [\dot{m_{f}}( \phi_f^{HO} - \phi_f^{U} )]_{explict}

    
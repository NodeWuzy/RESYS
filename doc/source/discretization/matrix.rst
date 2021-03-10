=================================
矩阵与方程
=================================

为了对物理场进行整场联立求解，需要利用方程构建出线性代数方程阻。
对于线性方程，可以直接通过联立求解线性方程得到。
而对于非线性的方程，求解通过线性化后的线性代数方程组，以及非线性外迭代求解。


本小节介绍RESYS程序中使用的稀疏矩阵格式。
在RESYS程序的初步开发中，使用的稀疏矩阵格式为 ldu 稀疏矩阵格式， 这种稀疏矩阵格式是计算流体力学软件 OpenFOAM 中使用的稀疏矩阵格式，比较适合用于流动传热的程序的开发。
尽管openFOAM中有多种该稀疏矩阵格式的求解器，但是在RESYS程序中目前使用的矩阵求解器局限于之前自行开发的求解器：

-  :class:`Jacobi`  : 最基本的雅可比迭代算法，其收敛较慢。但是易于并行，且计算量小，求解速度较快，在某些情况下适合用于能量对流扩散方程的求解。

-  :class:`PLU` ：   选主元高斯分解直接求解法，在矩阵较小时速度较快。

-  :class:`GMRES` ： 工程计算中常用的求解非对称矩阵的Krylov子空间迭代法。由于程序中目前未实现并行与预处理，因此效率较低。

-  :class:`CG` ：    工程计算中常用的求解对称矩阵的Krylov子空间迭代法。CG算法局限于求解对称矩阵，但是具有非常快的求解速度，适合用于求解 ``压力泊松方程`` 。


在后期的开发中，可以考虑研究实现更高效的矩阵求解器或者使用OpenFOAM中的求解器或者使用Petsc等稀疏矩阵求解库。

lduMatrix矩阵格式
=========================
关于ldu格式稀疏矩阵比较好的参考资料是关于OpenFOAM的书籍：《The Finite Volume Method in Computational Fluid Dynamics - *An Advanced introduction with OpenFOAM and Matlab*》。
在这里会复述其中的一部分内容，并且介绍这种稀疏矩阵格式在RESYS程序中的实际应用。

一个介绍ldu稀疏矩阵格式的很好的例子如下图所示。矩阵A被分解为对角元部分 ``diag`` 、上三角部分 ``upper`` 、下三角部分 ``lower``。
这三个部分使用三个单独的数组存储。

对于存储对角元对角元部分的数组 ``diag`` ， 矩阵第i行的对角元存储在数组 ``diag`` 第i个位置上。

对于存储上三角部分的数组 ``upper`` 和 存储下三角部分的数组 ``lower`` ，这两个数组具有相同的长度。
并且对于同一数组序号i上存储的两个元素，它们位于矩阵关于对角元对称的位置。

对于非对角元在矩阵的位置上的描述，通过两个整型数组 ``lowerAddr`` 和 ``upperAddr`` 来描述。
对于存储上三角部分的数组 ``upper`` ，其第 ``i`` 个元素在矩阵位置上的第 ``upperAddr[i]`` 行 ，``lowerAddr[i]`` 列。
对于存储下三角部分的数组 ``lower`` ，其第 ``i`` 个元素在矩阵位置上的第 ``lowerAddr[i]`` 行 , ``upperAddr[i]`` 列。


.. image:: /_static/lduMatrix.png
   :height: 390px
   :width:  500px
   :align: center

在RESYS程序中一个ldu矩阵是一个 ``lduMatrix`` 类的对象。在该类中，定义了lduMatrix求解器的父类，任何求解器的定义，均需要继承该类。
另外在 ``lduMatrix`` 类中定义了求解器的指针 ``lduMatrix_solver * solver`` 。 当需要对ldu矩阵构建求解器时，需要将构建好的求解器的指针赋给 ``solver`` 指针。

.. code-block:: c++

	class lduMatrix {
		matrixs * norm_matrix = NULL;
	public:

		// 求解器
		class lduMatrix_solver {

		public:
			virtual double * solve(lduMatrix * matirx, double * b, double * ans) = 0;
		};

		// 预处理算子
		class lduMatrix_preconditioner {
		public:
			virtual void construct(lduMatrix * matirx) = 0;
			virtual double * precondition(lduMatrix * matirx, double * b, double * ans) = 0;
		};
    
        ...
        }



-----------------------------
 lduMatrix for volume field
-----------------------------

对于定义在控制体上的物理场( :class:`volume field` ) , 每个控制体上的物理场通过接管与该控制体相邻的控制体的物理场相联系起来。
而边界上的物理场，用于指定边界条件，并不需要在矩阵中进行求解。所以时间相关控制体的数目不会影响到矩阵的维数，以及待求解的未知向量 ``x`` 的维数。
因此，对于 :class:`volume field` 其矩阵的维数为内部的控制体的数目 :class:`internal volume number` 。
而由于这些控制体通过内部接管相连，所以 数组 ``upper`` 和 ``lower`` 的维数为内部接管数目 :class:`internal junction number` 。
而且这些数组定义的顺序和接管的排序方式一致，因此数组 ``lowerAddr`` 和 ``upperAddr`` 分别等于描述接管连接的接管两端控制体编号 ``neigh`` 和 ``owner``。

在RESYS程序中，需要定义在 :class:`volume field` 的ldu矩阵时，通过使用函数 ``define_ldumatrix_on_volumes`` 来定义。


-----------------------------
 lduMatrix for junction field
-----------------------------

对于定义在接管上的物理场( :class:`jucntion field` ) 其大小为接管的总数(包括边界接管)。
由于动量方程在接管上使用有限差分形式离散，因此需要在接管上建立有限差分方程。
对于隐式的动量方程，需要使用矩阵对动量方程进行求解，这就要求RESYS程序能够在接管上建立ldu稀疏矩阵。
在RESYS程序中，在在接管上定义lud稀疏矩阵通过调用函数 ``define_ldumatrix_on_junctions`` 来定义。

对于 :class:`jucntion field` ，边界接管上的物理场也会参与求解。因此稀疏矩阵的维数是接管的总数。
需要考虑边界接管的物理场的求解是因为在RESYS程序中，对于边界上的接管的控制体的划分的方式是将与之相邻的内部控制体的一半划归为边界接管的控制体。
所以 :class:`jucntion field` 的稀疏矩阵的 ``diag`` 的维数为接管的总数。
而对于 :class:`jucntion field` ，其矩阵 ``upper`` 和 ``lower`` 的大小以及其位置信息描述数组 ``lowerAddr`` 和 ``upperAddr`` 的建立比 :class:`volume field`  更为困难，
这里不作相应的描述。




























































































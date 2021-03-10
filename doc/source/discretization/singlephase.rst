=======================================
单相流动方程的建立
=======================================

单相绝热流动方程
--------------------

在RESYS程序中，单相绝热流动模型的建立在类 ``single_phase_isothermal_flow`` 中，该类是一个流动模型求解类，因此该类继承于RESYS程序中定义流动模型的基类 ``model`` 。

单相绝热流动模型不进行传热计算，事实上，该流动计算模型开发的目的是做RESYS程序数值离散格式的验证以及收敛性的分析。
在后续的开发工作中，需要开发非绝热的单相流动方程的求解模型时可以继承该类实现代码重用。

为了实现算法的通用性，单相绝热模型使用SIMPLEC算法求解。
在最初步的对于单管问题的算例的计算中表明SIMPLEC算法表现出比SIMPLE算法更为优良的稳定性。
并且在实际的计算中表明，动量方程的对流项使用中心差分时，在网格较细的时候会不稳定。
为了提高算法的稳定性，实现了动量方程对流项的迎风差分。
另外，对于边界接管上的对流项，实际的计算表明，当忽略掉边界接管的对流项会提高程序的收敛特性。


单相绝热流动模型通过求解动量守恒方程和质量守恒方程来获得流场和压力场以及密度场：

- ``质量守恒方程``

.. math::
   \frac{\partial \rho A}{\partial t} +  \frac{\partial \rho u A}{\partial x}  = 0

- ``动量守恒方程``

.. math::

   \frac{\partial \rho u A}{\partial t} +  \frac{\partial \rho u^2 A + p A }{\partial x}  
   = \widetilde{p} \frac{\partial A}{\partial x} - F_{wall fraction} + \rho A g_x  


当使用SIMPLEC算法求解上面的方程组时，其过程是一个校正迭代的过程。
在下面会介绍上面方程的在RESYS程序中的求解过程，在最后会给出一个简单的单管算例的计算结果。
单相绝热流动模型的求解在类 ``single_phase_isothermal_flow`` 的 ``solver`` 函数中 ， ``solver`` 函数是重载了基类 ``basemodel`` 的虚基函数   ``solver`` 。
因此可以对每个求解器定义不同的求解函数 ``solver`` 然后在输入文件中调用。

类 ``single_phase_isothermal_flow`` 的 ``solver`` 方法如下所示：

.. code-block:: c++

    //=========================================
    // 单相绝热模型求解 , 使用simplec算法
    //=========================================
    void  resys::model::single_phase_isothermal_flow::solver() {

        using namespace globleParams;
        double time = 0;
        int iteration_times = 0;

        //--------------------------------------
        // 在本算法中，各个回路的水力学单独计算
        // 并且求解是独立的，所以各个回路可以并行
        // 计算
        //--------------------------------------
        forALL(component::loop_vector, loopI) {

            do  {

                iteration_times++;
                
                //---------------------------------
                // 复制流场物理量
                //---------------------------------
                field::copy_junction_field<field::scalar>(
                    velocity[loopI], 
                    velocity_saved[loopI] , 
                    component::loop_vector[loopI]
                );

                //--------------------------------
                // 建立压力边界条件
                //---------------------------------
                field::apply_pressure_boundary_condtion(
                    pressure[loopI],
                    time,
                    component::loop_vector[loopI]
                );

                //---------------------------------
                // 求解状态方程
                //---------------------------------
                liquid_property::get_single_phase_thermal_property(
                    property_var[loopI],
                    component::loop_vector[loopI].thermal_property,
                    int(loopI)
                );

                //---------------------------------
                // 密度进行赋值
                //---------------------------------
                forALL(property_var[loopI], i) {
                    density[loopI][i] = property_var[loopI][i].density;
                }
                
                //---------------------------------
                //对密度进行插值
                //---------------------------------
                face_density[loopI] =
                    interpolation::liner_interpolation().interpolate(
                        density[loopI],
                        face_density[loopI],
                        component::loop_vector[loopI]
                );

                //---------------------------------
                // 右端项归零
                //---------------------------------
                field::set_junction_field_to<field::scalar>(
                    rhs_meq[loopI],
                    component::loop_vector[loopI],
                    0.0
                );
                
                //---------------------------------
                // 矩阵清零
                //---------------------------------
                v_matrix[loopI]->set_all_element_to_value(0.0);

                //---------------------------------
                // 计算压力差值项
                //---------------------------------
                rhs_meq[loopI] =
                resys::explict_operater::face_differ(
                        pressure[loopI],
                        rhs_meq[loopI],
                        component::loop_vector[loopI]
                );

                //---------------------------------
                // 计算重力项 
                //---------------------------------
                rhs_meq[loopI] =
                resys::explict_operater::gravity(
                        face_density[loopI],
                        rhs_meq[loopI],
                        component::loop_vector[loopI]
                );

                //----------------------------------
                // 计算速度插值系数
                //----------------------------------
                interpolation_v->make_weight(
                    face_density[loopI],
                    component::loop_vector[loopI]
                );

                //---------------------------------
                // 计算摩擦项
                //---------------------------------
                apply_friction_term(int(loopI));
                
                //---------------------------------
                // 计算形状阻力项
                //---------------------------------
                apply_form_loss_term(int(loopI));
                
                //---------------------------------
                // 计算对流项,迎风格式
                //---------------------------------
                apply_upwind_momentum_convection_term(int(loopI));

                //---------------------------------
                // 计算非稳态项
                //---------------------------------

                //---------------------------------
                // 求解动量方程
                //---------------------------------
                v_matrix[loopI]->solve(velocity[loopI], rhs_meq[loopI]);

                //---------------------------------
                // 进行压力场校正
                // 计算质量守恒方程残差
                //---------------------------------
                compute_massflux_error(int(loopI));

                //---------------------------------
                // 构造压力校正矩阵
                // 求解压力校正值
                //---------------------------------
                solve_simplec_pressure_correction_equation(int(loopI));

                //---------------------------------
                // 修正速度场和压力场
                //---------------------------------
                make_correction(int(loopI));
                
            } while (! converge( int(loopI) ));
                
            cout << "iteration time "<< iteration_times << endl;
        }
    };

目前为止，其只是一个简单的求解稳态问题对流方程的SIMPLEC算法。
当然单相绝热流动求解器的开发还未完成。
目前的开发的求解器的目的是用于RESYS程序简单算例的验证以及数值离散方法的研究。
下面会介绍方程中一些项的离散格式。

和RELAP5程序类似，求解器对于每个回路分开求解，并且每个回路互相独立。
因此在求解程序的最开始的位置可以看到求解器对于每个回路循环独立执行 ``forALL(component::loop_vector, loopI)`` 。
``loopI`` 是循环体中求解的回路的序号。

- 摩擦项的离散

其次比较值得注意的是求解器中，``摩擦项`` 的离散方式使用线性化后的隐式形式以加速收敛。摩擦项的离散在函数 ``apply_friction_term(int(loopI))`` 中实现。
对于某一接管上的摩擦项的离散公式如下：

.. math::
    F_{wall fraction} / A = \frac{ f \rho_{L} D_L}{2 d_{hL}} |v_{f}^{L}| v_{f}^{L} + \frac{ f \rho_{R} D_R}{2 d_{hR}} |v_{f}^{R}| v_{f}^{R} 

对上面的公式使用线性化之后的结果:

.. math::

    F_{wall fraction} / A   = [\frac{ f \rho_{L} D_L}{2 d_{hL}} |v_{f}^{L}| v_{f}^{L} + \frac{ f \rho_{R} D_R}{2 d_{hR}} |v_{f}^{R}| v_{f}^{R}]_{explict} 
                          
    + [ \frac{ f \rho_{L} D_L}{d_{hL}} |v|_{f}^{L} v_{f}^{L} + \frac{ f \rho_{R} D_R}{ d_{hR}} |v|_{f}^{R} v_{f}^{R} ]_{implict}  
                          
    - [ \frac{ f \rho_{L} D_L}{d_{hL}} |v|_{f}^{L} v_{f}^{L} + \frac{ f \rho_{R} D_R}{ d_{hR}} |v|_{f}^{R} v_{f}^{R} ]_{explict}  

尽管摩擦项的离散格式还有待商榷，但是在程序中对其进行修改并非一件困难的事情。
在上面的有限差分公式中，摩擦项的计算分为接管owner控制体(与接管入口相连)侧和出口侧控制体(与接管出口相连)两部分。
对于上面公式中的第二项和第三项，使用的速度的绝对值是从速度插值函数类的 ``interpolate_abs`` 函数中获取的速度绝对值，其是与控制体相连的所有接管上的速度的加权平均。
对于上面公式中的第一项，使用的仅仅是使用线性插值速度插值函数类  ``interpolate`` 获取的控制体中心速度的绝对值。

在有的时候，上面的公式可能会出现一些问题，比如在分支部件中，控制体两端入口出口的速度大小相等方向相反时，上面的公式中该控制体部分的收敛的结果是0，这与物理事实并不相符。
但是假如将对于上面公式中的第一项的速度绝对值也使用 ``interpolate_abs`` 函数中获取的速度绝对值，就可以计算出符合物理事实的结果。这时上面的公式变为：

.. math::

    F_{wall fraction} / A   = [ \frac{ f \rho_{L} D_L}{d_{hL}} |v|_{f}^{L} v_{f}^{L} + \frac{ f \rho_{R} D_R}{ d_{hR}} |v|_{f}^{R} v_{f}^{R} ]_{implict}  
                          
    - [ \frac{ f \rho_{L} D_L}{ 2 d_{hL}} |v|_{f}^{L} v_{f}^{L} + \frac{ f \rho_{R} D_R}{ 2 d_{hR}} |v|_{f}^{R} v_{f}^{R} ]_{explict}  


对于控制体出口和入口接管均为同向流动的情况上面两个公式会给出相同的结果。
在后面的开发中，需要在控制体的接管涉及到相反方向流动的情况的时候研究哪一种能给出更符合物理事实的结果。

-对流项的离散 




下面给出了模拟一个单管流动的输入文件(暂定)：

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <problem description = "A one pipe flow problem" >
    <!--the globle settings block set up the globle parameters to be used in the input-->
    <globle>
        <model_type> single_phase_isothermal_flow </model_type>
    </globle>
        
    <!-- the flow model block set up the flow models to be used in the simulations -->
    
    <flowmodel>
        <modle  type = "single_phase_isothermal_flow"/>  
    </flowmodel>
    
    <!--
        the stateEquation block set up the equation of state 
        used in thermal hydrodynamic calculation
    -->

    <stateEquation>
        <barotropic
        name            =   "barotropic"
        p_0             =   "1.0e5"
        rho_0           =   "1.0e3"
        a2              =   "1.0e7"
        />
    </stateEquation>
    
    <!-- 
    The component block specifies the components 
        to be used in the simulations 
    -->
    
    <component>
        
        <pipe    
        name              =     "pipe1"   
        start_position    =     "0  0  0"
        orientations      =     "1  0  0"
        pipe_length       =     "2.0"   
        n_elems           =     "10"
        pipe_area         =     "1.0E-4"
        pipe_Dh           =     "0.01"  
        friction          =     "0.1"
        Hw                =     "0.0"
        />
        
        <timeDependentVolume
        name     =   "inlet" 
        input    =   "pipe1.in"
        p_bc     =   "1.0E5"
        />
            
        <timeDependentVolume
        name     =   "outlet"
        input    =   "pipe1.out"
        p_bc     =   "1.05E5"
        />
        
    </component>

    <loop>
        <loop 
            referent          = "pipe1"  
            working_substance = "barotropic" 
        />
        
    </loop>

    <!-- 
        The executioner block specifies the 
        executioner that will be used in the simulation 
    -->
    
    <executioner>
    </executioner>
    </problem>

    





.. _xml输入指南:

===================
xml输入指南
===================

RESYS程序通过读取一个用于描述描述模拟问题的xml输入文件来建立热工水力模型以及边界条件和初始条件。 `xml <https://www.w3.org/XML/>`_ 是一种树状的层级文件格式。RESYS程序的输入主要包括六个部分：

1. 全局变量
2. 流体状态方程
3. 材料热物性
4. 热结构
5. 水力学部件
6. 求解设置

因此在RESYS程序读取的xml文件中会有六个输入模块，每个输入模块为根元素下面的一个子元素，每个子元素描述上面列表中的一部分。
对于特定的问题，某些部分可以被省略掉，比如在没有热传导计算的问题中，热结构输入模块可以不用输入，但是关于流动传热计算的水力学部件部分是默认需要的。
RESYE程序输入使用国际单位制。输入单位制的统一避免了在模型中使用不同的单位而可能引入的错误，并且方便输入文件的检查。
RESYS程序的输入文件的结构如下：

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <problem description = "..." >
        
        <global>
           <!-- 这个输入模块内部输入全局设置 -->
           ...
        </global>

        <material>
           <!-- 这个输入模块内部输入固体材料物性 -->
           ...
        </material>
         
         <flowmodel>
            <!--这个输入模块用于定义流动模型-->
         </flowmodel>

        <stateEquation>
           <!-- 这个输入模块内部输入流体热物性 -->
           ...
        </stateEquation>

         <function>
            <!--这个输入模块用于定义函数-->
         </function>

        <component>  
          <!-- 这个输入模块内部输入水力学部件 -->
          ...
        </component>
        
        <loop>
         <!--这个输入模块内部输入回路的设置-->
         ...
        </loop>

        <heatStructure> 
          <!-- 这个输入模块内部输入热结构信息 -->
          ...
        </heatStructure>
        
        <executioner>
          <!--这个输入模块内部输入求解设置-->
          ...
        </executioner>

    </problem>

.. toctree::
   :maxdepth: 1
   :caption: 输入指南:

   component.rst
   thermionic.rst




    

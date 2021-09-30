.. RESYS documentation master file, created by
   sphinx-quickstart on Mon Jun 29 21:48:55 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

=================================
RESYS 核反应堆系统分析程序
=================================

RESYS(Reactor System Analyse Code)是一款用于核反应堆系统瞬态行为分析的热工水力程序。
其设计目标是实现一款适用于多种反应堆系统的系统分析程序。
本程序使用面向对象的编程语言C++编写，由中国原子能科学研究院（CIAE）软件室开发和维护。
目前开发的是RESYS程序的第一个版本，考虑到程序的复杂性（同类程序往往达到数万行），第一个版本的目标设定为模拟反应堆内单相流动换热。
这对于大多数的类型的液态金属冷却快堆和气冷堆是足够的。
在后续的版本中，考虑加入两相流动模型（如漂移流模型、两流体模型），以模拟压水堆和沸水堆系统的瞬态工况。
而第一个版本主要侧重于程序的结构设计，用户体验良好的输入设计，以及确保程序具有足够的可扩展性，以适用于不同类型的反应堆系统的安全分析。

RESYS程序使用控制体建模的系统建模方法，通过对反应堆系统划分控制体，并利用有限体积法对控制体列出质量、动量、能量守恒方程，
以模拟核反应堆系统内部的流动与换热现象。RESYS程序的系统建模方法和relap5很类似，熟悉relap5的使用者可以很容易掌握使用本程序。



.. toctree::
   :maxdepth: 1
   :caption: Contents:

   equaltion/index.rst
   component/index.rst
   discretization/index.rst
   xml_input/index.rst
   algorithm/index.rst
   develop/index.rst
   example/index.rst

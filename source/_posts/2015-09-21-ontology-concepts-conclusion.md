---
layout: post
title: "Ontology本体基础知识总结整理"
date: 2015-09-21 12:00:00
comments: true
categories: 
- Document
tags:
- Knowledge Base
- Ontology
- 基础知识
---


#### 本体的概念

共享概念模型的明确的形式化规范说明。包括四个主要的方面：

- 概念化（conceptualization）：客观世界的现象的抽象模型；
- 明确（explicit）：概念及它们之间联系都被精确定义；
- 形式化（formal）：精确的数学描述；
- 共享（share）：本体中反映的知识是其使用者共同认可的。

<!-- more -->

#### 本体构成要素

- 类：集合（sets）、概念、对象类型或者说事物的种类；
- 关系：关系代表了在领域中类之间的交互作用。形式上定义为n 维笛卡儿乘积的子集： R ： C1 ×C2×⋯×Cn 。 如：子类关系( subclass-of)；
- 函数： 函数是一类特殊的关系。在这种关系中前n - 1 个元素可以惟一决定第n 个元素。形式化的定义如下： F ： C1 ×C2 ×⋯×Cn-1→Cn 。例如Mother-of 关系就是一个函数，其中Mother-of ( x ， y) 表示y 是x 的母亲，显然x 可以惟一确定他的母亲y；
- 公理：公理代表永真断言，比如概念乙属于概念甲的范围；
- 实例：实例代表元素，即某个类中的第一个对象；

#### 专家系统

* 知识库：
* 推理机：本体按照是否具备推理功能可以分为轻量级本体、中级本体和重量级本体；技术构成库应该属于可以识别一阶谓词逻辑表达式的中级本体

#### 本体语言

- 本体语言目前成为标准的是OWL；
- Cyc和loom两种语言具有较强的推理能力；

#### 领域本体的知识库构建

- 本体构建原则：清晰、一致、可扩展性、编码偏好程度最小、本体约定最小；

- 两种模式：

  1. 利用现有文献和领域专家使用手工的方式创建概念关联；
  2. 将已有的叙词表改造成本体，或者采用学习机制，进行自动或自动化的本体构建。
  3. 本体手工构建方法：骨架法、企业建模法、Methontology、KACTUS、循环获取法、IDEF-5、七步法；
  
    七步法：

     - 确定本体的专业领域与范畴；
     - 考查复用现有本体的可能性；
     - 列出本体中的重要术语；
     - 定义类和类的等级关系；
     - 定义类的属性；
     - 定义属性的分面(Facets)；
     - 创建实例。

  4. 领域本体知识库架构三个层次：

     - 表示层：表示层是用户与系统交互的接口，用户通过浏览器或其他界面访问系统，用户界面负责接收查询请求， 并将服务端的检索结果回显给用户； 
     - 业务逻辑层：业务逻辑层为主要应用逻辑层，实现系统知识的检索。它由本体管理组件、语义分析组件、推理引擎、查 询组件和信息获取组件5个部分组成；
     - 数据层：数据层包括3个部分：本体库、资源描述库、资源数据库。它是知识库的存储介质，创建并提炼出结构化 的知识本体，是知识检索的直接来源；

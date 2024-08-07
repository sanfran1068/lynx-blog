---
layout: post
title: "形状文法简介"
date: 2017-10-08 12:00:00
comments: true
categories: 
- Research
tags:
- 形状文法
- 基础知识
---

形状文法（shape grammar）最早由George Stiny和James Gips在1972年提出，是一种用带符号的形状作为基本要素，用语法结构分析和产生新的形状的设计推理方法，是一种以运算规律为主的设计方法。最早运用于绘画、雕塑和建筑设计领域，后被推广到工业设计领域。我的毕设开题之后，是实验室和阿里一起合作的项目中的一个系统实现，其中大量地使用到了形状文法的方法，所以有必要对该研究进行一些了解和知识点的整理。

<!-- more -->

#### 概念

更加简洁的概念：形状文法是一种计算机辅助设计方法，它可以按照人们的设计思想和要求，依据一定的规则产生新的形状。形状是指有限直线段的组合，并且带有符号标记，以指示形状生成的方向；规则是指对形状进行改变的规定。广义地来讲，形状文法是对已有产品特征的分析、描述和归纳；再现和发展原有产品特征，形成新的风格

- 形状文法的定义：

    形状文法定义为四元组，即SG＝(S, L, R, I)。其中，Ｓ为形状的有限集合；Ｌ为符号的有限集合；Ｒ为规则的有限集合，规则的基本形式为α → β，α是一个带符号形状，α∈(S, L)+，β也是一个带符号形状，β∈(S, L)，(S, L)＋是一个形状和符号所组成的集合，（S, L）则是（S, L）+∪（SΦ, L）；Ｉ为初始形状，I∈(S, L)+．一个形状文法定义的形状集合称为语言，这个语言包含所有的由形状文法生成的无符号形状或带符号形状，其中的每一个形状都是通过应用形状规则从初始形状派生出来的，如下图所示：
    
    ![](https://img.alicdn.com/imgextra/i3/O1CN01x8tVcl1G7qyHzKkRn_!!6000000000576-0-tps-1566-1204.jpg)
    
    基础的形状文法规则有生成性规则和衍生性（修改性）规则，衍生性规则是在生成性规则的基础上进行更多参数和更多步骤的规则变化，能够生成更加复杂的纹样和图案。

- 形状文法应用流程：

    以纹样的形状文法生成为例来说，最基本的步骤分为三步：
        
    1. 纹样研究，对传统纹样及相关文化背景进行研究以锁定目标纹样。目标纹样应为传统纹样的代表，能够反映特定的文化特色。再针对目标纹样进行特征分析，包括纹样主题纹样构图规律、骨骼形式、纹样元素组织方式等信息。
        
    2. 形状演变，对第一部分得到的典型图形运用形状文法的规则进行演变，生成基本图形单元。再对基本图形单元进行派生，以得到多种创新纹样元素。
        
    3. 方案生成，设计师对派生出来的多种纹样元素进行筛选，再以纹样设计文法为约束对纹样元素进行合理组织、重构，以生成新的纹样设计方案。
        
#### 形状文法实施过程

- 问题定义：即对即将要使用形状文法的客体进行研究问题的确定，一般从逻辑推理的方法得出对问题的定义；

- 初始形状确定：

    1. 收集样本：选定某个领域的客体实例作为研究对象，广泛收集该领域各个发展阶段的图片，将这些图片进行整理、分类，去除类型接近的图片，找出代表性的图片，将其制作成可供调查的样本，并以一定的方式编号（labeling）。
    
    2. 专家访谈：邀集相关的专家进行访谈，一般应包括资深设计师和深度使用者。访谈的主要目的首先在于对影响该纹理特征进行分类，然后对各类特征的重要程度做出主观评价，以确定各个因子的权重．再根据纹理分类和专家的直观经验，列举出该领域实例主要的可识别特征，为进行下一步设计形态分析做准备。
    
    3. 设计形态分析：设计形态分析法是由荷兰学者Ware，具体过程是：
        
        - 将选取的产品样本图片和专家列举的产品显性特征制成调查表，的受试者选择面要广，包括专家、设计师、一般用户等，年龄、性别和职业要分布合理。
        - 由受试者将产品样本一一对照各个形态特征进行判断，以“强”、“弱”、“无”作为标准，并对应以不同分数。
        - 对调查结果进行统计分析，判断并提取产品的风格特征。

- 形状规则的确定：

    - 生成性规则：生成性规则的形式可分两种：**添加**，即从无到有；**置换**，以一个完全不同的全新形态代替原形态．

    - 修改性规则：“修改”就是指对原有形态的改变、变形、调整．由于生成性规则是对纹理本质的“变异”，因而修改性规则应侧重于“遗传”，否则纹理的显性特征变得面目全非，品牌的延续将无从谈起。例如，可以使用仿射变换（affine transform）进行变形，或者对整体轮廓曲线采用贝塞尔曲线（Bezier curve）进行描述并进行非精确性变形。
    
- 设计要求与约束规则：
    
    在完成以上两项初始形状和文法规则的确立之后，还要对于已确定问题的边界和约束进行规定。
    
#### 目前形状文法落实进展：

- 形状生成：Tranglify算法生成随机背景

- 形状布局：通过布局风格（平铺/随机）、布局区域（全局/环绕）、形状旋转和形状缩放的python代码实现

- 颜色渲染：图片颜色整个色盘选择替换
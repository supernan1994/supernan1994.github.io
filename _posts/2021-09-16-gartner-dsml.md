---
layout: post
title:  "2021年Gartner数据科学与机器学习(DSML)平台魔力象限报告解读"
date:   2021-09-16 00:00:00
categories: marketing
comments: true
image: 2021-09-16-gartner-dsml/cover.png
tags: [marketing]
---

Gartner是全球最具权威的IT研究与顾问咨询公司，Gartner魔力象限（Magic Quadrant，简称MQ）是一种针对企业级IT市场的分析方法，用来形象化的阐述公司间的实力及差异。谷歌、微软、甲骨文、IBM等科技巨头都非常在意Gartner魔力象限的评价，并以上榜为荣；同时，世界五百强的CIO们在采购技术产品时，也大多将Gartner魔力象限作为一个重要评价依据。   
今年3月Gartner发布了2021年[Data Science and Machine Learning Platform Magic Quadrant](https://www.gartner.com/document/code/467320)（简称DSML MQ），8月我听了一场由Gartner研究总监孙鑫主持的[Webinar](https://www.gartner.com/en/webinars/4003705/get-insights-from-the-gartner-magic-quadrant-for-data-science-machine-learning)[[Slides](https://www.brighttalk.com/resource/core/357465/aug25jsun_779829.pdf)]。  

这篇文章主要是Webinar的学习笔记，总结一些核心观点，也包含了一些我个人的理解。文章里没有太多涉及对某个Vendor的评估，更多是对整个行业的研讨。  

## 1. DSML平台定义
拥有一个核心产品实现全生命周期的数据科学活动。同时，产品需要具备集成第三方（包括开源）的组件、框架的能力。  

### DSML平台3个基本特征
- 平台能够提供构建DSML解决方案的能力：具体指提供数据处理、预置模型等能力。【即：能搭】
- 能够将解决方案集成到业务流程中：即提供ModelOps平台，将模型运维、集成产品化管理。【即：能用】
- 赋能不同背景的数据分析人才：能够满足Data Scientist到Citizen Developer的需求，适应企业人力配置。主要指平台的低代码或无代码属性。【即：所有人都能用】

### DSML平台核心功能
<div align="center"><img width="80%" height="80%" src="2021-09-16-gartner-dsml/WX20210916-160711@2x.png"/></div>
产品能力评估在魔力象限评估体系中权重比较高。分析师在这里没有进行过多描述。  

❗️商业机器学习落地是一门非常复杂的工程问题，涉及到非常多的细分领域，而且每年都在发展。这份报告中更多是站在商业、市场、产品角度给出的分析，如果你想从工程、学术角度了解机器学习平台架构，强烈建议看一下UC Berkeley的[Full Stack Deep Learning](https://fall2019.fullstackdeeplearning.com/)课程。

<div align="center"><img width="80%" height="80%" src="2021-09-16-gartner-dsml/CleanShot 2020-07-06 at 19.11.29@2x.png"/></div>

由于问题的复杂性和用户的多样性，DSML平台根据使用场景分化出4种不同的流派，厂商各有侧重。这四个场景分别是：
- 用于数据和业务探索
- 平民数据科学
- 专家级模型研发
- 模型运行与运维


## 2. DSML MQ评估标准和过程中得到的经验
### 评估标准

<div align="center"><img width="80%" height="80%" src="2021-09-16-gartner-dsml/cover.png"/></div>

MQ的纵轴和横轴分别代表执行力和前瞻性。
- 执行力关注当下。涉及到用户体验、运营、产品能力、公司生存状态、定价、市场能见度、大客户服务能力等方面。
- 前瞻性关注未来。涉及到市场理解、产品战略、创新能力、市场战略、行业战略、销售战略、地域战略等。  

换个更接地气的说法，MQ评估的是一个基本的商业逻辑：企业能不能把NB吹的别人都信了（前瞻性），并且能做出来（执行力）。  

### 过程中得到的经验
- 在疫情期间，DSML领域表现出比较强的韧性，营业额产生了正向的增长。
- 在海外，目前on-premise和cloud部署DSML平台的比例差不多，cloud增长较快，2016'30%，2024'67%。
- 蛋糕太大，很多vendor开始合作。

## 3. DSML平台的主要趋势

### 主要趋势
<div align="center"><img width="80%" height="80%" src="2021-09-16-gartner-dsml/WX20210916-183726@2x.png"/></div>

这些趋势是Gartner通过跟客户和投资人在Inquiry中逐渐总结出来的。

- 增强型DSML：自动化或增强数据处理、模型构建、在线服务的各项工作。大多数供应商关注这个方向，小厂做的更好。
- 多人协作：越来越多的企业在组建机器学习团队，多人协作能力是刚需。
- ABI/DSML能力结合：让会用BI的人掌握通过DSML分析数据的能力。
- Xops（DataOps，MLOps，ModelOps）：利用DevOps的最佳实践实现效率和规模经济，并确保可靠性、可复用性，减少技术的重复并实现一些自动化。
- 混合云/随处运行：容器化、分布式部署、边缘设备运行。
- 决策智能：在自动化共享的治理环境下，提供企业级的决策流程。
- 产品的开放性：和开源工具的互操作性和集成能力。
- XAI：可解释性AI，通过更好的把控和治理AI来降低风险。

### 交付模式的多样性
<div align="center"><img width="80%" height="80%" src="2021-09-16-gartner-dsml/WX20210916-195414@2x.png"/></div>

通常有3种交付模式：自建、购买、外包。3种各有优劣势，很多成熟企业会采纳混合的交付模式。

<div align="center"><img width="80%" height="80%" src="2021-09-16-gartner-dsml/WX20210916-201752@2x.png"/></div>

搭建好DMSL平台后，企业需要考虑如何与现有的BI平台和自助式分析平台集成，通过building block方式实现组装式的分析。


### 哪些工作可以被自动化
<div align="center"><img width="80%" height="80%" src="2021-09-16-gartner-dsml/WX20210916-200632@2x.png"/></div>

全自动化的机器学习平台是不存在的，不同环节可被自动化的比例有所不同。

### 开源软件依然非常重要
<div align="center"><img width="80%" height="80%" src="2021-09-16-gartner-dsml/WX20210916-201243@2x.png"/></div>

开源软件和商用软件不完全站在对立面。MQ评估的厂商很多也是开源项目的发起人或重要贡献者。

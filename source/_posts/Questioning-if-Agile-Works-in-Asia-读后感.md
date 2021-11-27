---
title: Questioning if Agile Works in Asia 读后感
tags:
  - Scrum
  - 敏捷开发
categories: 杂谈
abbrlink: 37366
date: 2016-07-04 14:55:11
---
最近在infoQ上看到scrum.org的一位资深敏捷教练Joshua Partogi发表的关于敏捷开发的文章《Questioning if Agile Works in Asia》，结合目前自己工作中的敏捷实施现状，有感而发。

* [原文地址](https://www.infoq.com/news/2016/06/agile-asia)
* [译文地址](http://www.oschina.net/news/74785/why-agile-development-can-not-implemented-asia)

<!--more-->
文中提到了四点，总结如下：

1. 有人希望有人告诉自己该干什么，也有人总想指挥别人干事，整个组织工作井然有序。
> People expect to be told what to do and people want to tell other people what to do because that is how a system that is in order works.

2. 为了给别人留面子，亚洲人习惯于保留意见，而不是直接指出问题。
> Agile requires an ability to tell hard truths to those controlling the purse strings, which is decidedly difficult in Asian cultures that place a much higher premium on deference, respect, and "saving face" than is the case in the western cultures where agile techniques were pioneered.

3. 亚洲人更习惯于通过制定计划来使未来具有可预见性，然而对于软件开发这种复杂的、创造性的工作来说，可预见性这种事是不可能的。又想制定计划，又想搞敏捷，结果通常是：软件质量差、计划延期、资金浪费和士气低迷。
> People who are culturally attuned to predictability want to believe that they can predict the future. Their job is then to cause the future to come true by forcing the people and resources to make it happen. People who use Scrum have learned that predictability is impossible when complex, creative work like software development is done. The results are terrible: bad software, missed schedules, wasted money, and demoralized workers.

4. 亚洲的教育体制鼓励的是考高分，定级别，而不是为了尝试、自我发现和试错，而这些恰恰是敏捷实践的根本目的之所在。
> The Asian education system is all about high grades and ranks, not about experimenting, self- discovery and making mistakes, which is what Agility is all about.

字字珠玑，直指软肋。

我目前所在的公司，正好也在推行敏捷开发，然而几乎所有的同事表面不说，私下里却是对这种方式各种抱怨。（如果读到这里，你想给我推荐“不抱怨的世界”，那么请你不要继续读下去了）

这家公司的问题，几乎和第三点所说的完全契合。在敏捷管理中，所谓的“专家”们把“监控”和“问责”提升到了前所未有的高度，如果一个任务没有按时上线，或者上线后出现问题发生了回滚，那将是非常严重的事情，需要填写delay或者事故的原因分析，每15分钟汇报事故解决进展，领导层要自罚工资，事故发生的团队要扣KPI绩效……

如果一个团队发生了事故，那么所有团队都必须执行更加严格的上线审核制度，例如：对上线次数和时间做严格限制，撰写更多的上线自查文档，执行更严格的自动化代码检测（这个工作往往是非常耗时的）等。

正如 Joshua Partogi 所说的，他们希望事情能够变得更加“可控”，因此制定了种种规则来绑架敏捷开发的节奏，对犯错的责任人给予严惩，并希望杀鸡儆猴，让其他的人更加注意，以减少问题的发生。

# 然而这真的降低了问题发生率吗？

就我的观察来看，不仅没有降低，反而提升了。为什么？原因很简单：
1. 在工时挤压和繁重的流程面前，所有人都变得更加忙碌，于是“忙中出错”。
2. 在严苛的工时管理制度下，为了能够保证项目不delay，人们开始习惯于通过“最快”而非“最优”的方案解决问题。于是，你懂的，技术债务开始不断累积。而技术债务和问题发生率之间的关系是不言而喻的。

----

回到文中主题，为什么 Joshua Partogi 会对亚洲人有第一点和第二点的印象呢？其实并不是性格使然，而是一种“江湖策略”。

首先在亚洲国家，有着非常森严的等级制度，下级违抗上级在道义上是很严重的行为，在这种制度的暗示下，就产生了第一点：作为下级只要做好执行即可。

如果你思考多一点，觉得领导的思路存在一些问题，然后提了出来，那么在很多时候，这种行为会被解读为一种“冒犯”，所以为了“明哲保身”，人们宁可保留意见，也不愿指出来，这就形成了上述第二点。

同时，教育体制的影响也体现在了日常管理制度的建立上，人们习惯于通过某种“指标”来考核工作效率，注重“对与错”的判定，并理所当然地认为“罪与罚”是当务之急。

## 结语
在这样的意识形态下，推行敏捷开发确实困难重重。笔者认为，目前来看，这是无法逾越的屏障，如果你下定决心要在公司里推行敏捷开发，那么请审慎考虑当前公司的文化氛围，并合理制定敏捷的玩法。否则在以后的管理工作中，这将是噩梦的开始。

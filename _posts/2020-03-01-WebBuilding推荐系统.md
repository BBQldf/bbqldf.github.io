---
layout:     post
title:     web_building电影推荐
subtitle:   一种基于用户信息的协同过滤电影推荐算法
date:       2020-03-01
author:     ldf
header-img: img/post-bg-DewebBuild.png
catalog: true
tags:
    - 系统
---

# web_building

网站推荐系统
武汉大学国家网络安全学院暑期实训项目，最佳项目奖，最佳算法奖

环境：
Python-3.6
django
Html5
CSS3
Jquery

# 项目介绍
## 动机
现在电影资源是网络资源的重要组成部分，随着网络上电影资源的数量越来越庞大，设计电影个性化推荐系统迫在眉睫。所以本文旨在为每一个用户推荐与其兴趣爱好契合度较高的电影。本系统包含电影前端展示界面、电影评分板块、推荐算法的实现以及后端数据库的设计。
系统采用由grouplens项目组从美国著名电影网站movielens整理的ml-latest-small数据集，该数据集包含了671个用户对9000多部电影的10万条评分数据。首先将该数据集包含的全部文件经过筛选重组之后存储到建好的数据库中，并将数据集按一定比例划分为训练集和测试集，对训练集进行算法分析生成Top-K个性化电影推荐列表，然后在测试集上对算法进行评测，包括准确率（Precision）、召回率(Recall)两种评测指标。
## 项目组成
整个系统涉及到的编程语言包含Python、Html5、JQuery、CSS3以及MySQL数据库编程.用到的框架是Django重量级web框架，通过该框架连接系统的前、后端。
用户首先需要填写用户名、密码以及邮箱注册系统，然后才能登陆推荐系统.进入首页后会看到8个电影分类，包括恐怖片、动作片、剧情片等.用户需要给自己看过的电影进行评分，评分起止为0.5-5.0分，共10个分段.每评价一部电影就要点击一下提交按钮，将所评分的电影的imdbId号以及对应的评分存入数据库中.用户点击“推荐结果”按钮，系统就调用推荐算法遍历数据库所存数据，得出推荐列表之后将结果反馈给浏览器，同时调取数据库所存电影海报图片进行展示.用户点击自己登陆的昵称，会跳转页面显示自己已经评价过的电影.
## 项目架构
![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220807001345.png)

## 项目效果
1. 基于用户的预测
![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220807001417.png)
2. 基于电影的推荐
![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220807001437.png)

# 原理：
基于用户(user-based)的协同过滤推荐算法
基于用户的协同过滤算法是通过用户的历史行为数据发现用户对视频内容的喜欢(如视频点赞，收藏，内容评论或分享)，并对这些喜好进行度量和打分。根据不同用户对具有相同视频内容的态度和偏好程度计算用户之间的关系，在有相同喜好的用户间进行商品推荐。
在user-based协同过滤推荐算法中，user1和user3都收藏了product2和product3，通过对user的相似度计算，两者具有相近的距离。在后续的推荐过程中，user1独有的收藏视频product1和product4会被推荐给user3。

## 用户-电影评分矩阵的构建
我们从网络中获得到MovieLens数据集对用户-电影评分矩阵进行构建，数据集中包含610个用户对不同电影的10万个评分，分值为1至5，不同的用户通过电影ID相互联系
```python
"""
新建一个data字典存放每位用户评论的电影和评分, 如果字典中没有某位用户，则使用用户ID来创建这位用户,否则直接添加以该用户ID为key字典中
"""
file = open("./ml-latest-small/data.csv",'r', encoding='UTF-8')
data = {}
for line in file.readlines():
    line = line.strip().split(',')
    if not line[0] in data.keys():
        data[line[0]] = {line[3]:line[1]}
    else:
        data[line[0]][line[3]] = line[1]
```

## 寻找近邻用户
对于不同用户之间的相似性，常常通过简单的欧式距离进行计算[2]，我们将采用person系数，也称为皮尔森积矩相关系数，对不同用户间的相似性进行计算。对于不同的用户，person系数取值在[-1,1]之间，如果person系数接近于1，则表示两个用户之间具有强相关性，两者对电影具有相同的个人喜好。当person系数接近于-1时，两个用户间完全负相关，及表示两人对电影的喜好完全对立。Person系数为0时，表示两个用户对电影的个人喜好并没有太多的联系。

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220306174002.png)

## 推荐
针对我们将要推荐的用户，我们通过近邻用户间的评分进行预测，预测公式如下：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220306173956.png)

p为用户x对电影m的预测评分，s表示用户x的近邻用户，在本文中数量为10个。目标用户的近邻集合为Im。我们将最终得到的预测结果放到统一的集合中，并将预测结果进行降序排序，得到最终的电影推荐结果，并将结果输出。

## 数据库E-R设计
1. 用户E-R图
![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220807001511.png)
2. 电影E-R图
![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220807001608.png)
3. 推荐结果E-R图
![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220807001641.png)
4. 用户评分E-R图
![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220807001722.png)
5. 系统E-R图
![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20220807001800.png)


# 总结
本文工作完成了一个简单的个性化电影推荐系统，仍然还有很多地方不足需要进一步完善，主要有以下几个方面：
1. 采用的算法效率不够高。目前的算法是个经典算法，运行效率不高，速度很慢，后期可以考虑结合基于标签和基于内容的推荐算法进行推荐
2.本系统使用的是用户对电影的历史评分数据，这些数据都存在一定的稀疏性。登录用户看过的电影可能很少，这样就导致他和数据库中所存用户评过分的电影有较少的交叉项目，使得计算出的相似度很不准确。之后可以用更大的数据集，考虑遍历百万条评分数据来进行推荐




---
layout:     post
title:      Git指令整理
subtitle:   不适合阅读的整理的一些个人常用的 Git 指令
date:       2017-02-15
author:     BY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Mac
    - 终端
    - Git
---

>测试一下加图片和code格式（应该是可以的）

python 代码实现


    

```python
def generate_probability(epsilon, l, l1, l2):
    #array 'distance' and 'probability' have a one-to-one correspondence
    distance = np.zeros(int((l2 - l1)/2)+1)
    probability = np.zeros(int((l2 - l1)/2)+1)
    j = 0
    for i in np.arange(l1, l2 + 1, 2):
        distance[j] = i
        j = j + 1
    deltaL = (l2 - l1)
    x = sympy.symbols('x')
    y = (epsilon /(2* deltaL)) * sympy.exp(-epsilon * abs(x - l)/deltaL) + (sympy.exp((epsilon * l1 - epsilon * l)/deltaL) + sympy.exp((-epsilon * (l2 - l))/deltaL))/(2*deltaL)
    #y = (epsilon / 2* deltaL)*(-epsilon)^(-epsilon*(x-1)) + epsilon^(epsilon*l1-epsilon*l) + epsilon(-epsilon*(l2-l))/(2*(l2-l1)
    j = 0
    #refer "rationality"——sum(probability) = 1
    #find when the x in [l1,l2],each step is 0.1, the y counts the probability of x located in [l1,l2]   
    for i in np.arange(l1, l2 + 1, 2):
        probability[j] = sympy.integrate(y, (x, i, i + 2))
        #print("probilities=",probability[j])
        j = j + 1
    # print(probability)
    privacyLeakage = Privacy_leakage_comp(epsilon,l1,l2, l)

    return distance, probability, privacyLeakage
```



echo命令：

	echo "# 项目名" >> README.md
	git init
	git add README.md
	git commit -m "first commit"
	git remote add origin git@github.com:qiubaiying/项目名.git
	git push -u origin master



加入图片：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/%7BPF7L)AF4TKNMV%24X%7E1HENVF.png?token=AKXBU57ZULJJ6A3BMYCVCX3BPSSJA)

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211014161017.png?token=AKXBU57Q3BNSHOGNRV6JTJTBM7TCO)


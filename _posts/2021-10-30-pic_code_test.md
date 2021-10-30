---
layout:     post
title:      图片代码测试
subtitle:   利用typora和picGo上传代码和图片
date:       2017-02-15
author:     ldf
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - code
    - test
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

![截图-热点信息](https://raw.githubusercontent.com/BBQldf/PicGotest/master/%7BPF7L)AF4TKNMV%24X%7E1HENVF.png?token=AKXBU57ZULJJ6A3BMYCVCX3BPSSJA)
![截图-论文](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20211014161017.png?token=AKXBU57Q3BNSHOGNRV6JTJTBM7TCO)

![qq图片](https://raw.githubusercontent.com/BBQldf/PicGotest/master/QQ%E5%9B%BE%E7%89%8720210706110800.jpg?token=AKXBU563N6KGAMVNJJYR4FLBM7SXE)

![qq图片2](https://raw.githubusercontent.com/BBQldf/PicGotest/master/QQ%E5%9B%BE%E7%89%8720210706110721.jpg?token=AKXBU5ZDSYOEPQGCROOVSK3BPSUFK)

![qq图片2副本-测试中文路径](https://raw.githubusercontent.com/BBQldf/PicGotest/master/QQ图片20210706110721.jpg?token=AKXBU5ZDSYOEPQGCROOVSK3BPSUFK)


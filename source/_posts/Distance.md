---
title: iOS计算点到直线的距离以及与垂足的交点
---

## 1.问题描述
>假设有一点坐标P（x0,y0）,有一线段AB，A坐标（x1，y1），B坐标（x2，y2），求P点到AB线段或所在直线的距离d以及P点在直线上的垂足C（x，y）。
## 2.分析
首先需要将A，B两点坐标转换为直线方程的一般式Ax+By+C = 0，在根据距离公式计算
![距离公式.png](https://upload-images.jianshu.io/upload_images/1447375-ca8c2536138cb762.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下面是推算过程
![推算过程.png](https://upload-images.jianshu.io/upload_images/1447375-1318b93d0b69c326.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
参数计算：
A=y2-y1；
B=x1-x2；
C=x2*y1-x1*y2;
1.点到直线的距离公式：
d= ( Ax0 + By0 + C ) / sqrt ( A*A + B*B );
2.垂足C（x，y）计算公式：
x = (  B*B*x0  -  A*B*y0  -  A*C  ) / ( A*A + B*B );
y  =  ( -A*B*x0 + A*A*y0 – B*C  ) / ( A*A + B*B );

## 3.代码实现

```
func pedalPoint(p1: CGPoint, p2:CGPoint, x0: CGPoint) ->(Double, CGPoint) {
        
        let a = p2.y - p1.y
        let b = p1.x - p2.x
        let c = p2.x * p1.y - p1.x * p2.y
        
        let x = (b * b * x0.x - a * b * x0.y - a * c) / (a * a + b * b)
        let y = (-a * b * x0.x + a * a * x0.y - b * c) / (a * a + b * b)
        
        let d = abs((a * x0.x + b * x0.y + c)) / sqrt(pow(a, 2) + pow(b, 2))

        let pt = CGPoint(x: x, y: y)
        return (Double(d), pt)
    }
```

---
title: "力扣算法题小结"
date: 2026-04-25
#image: "cover.jpg"   # 封面图（可选）
math: true            # 如果有公式，记得开启
hidden: true
categories:
    - "数据结构算法"      # 这里填你想划分的板块名
tags:
    - "数据结构算法"
---

```C
#C++中定义长度不确定的二维数组
 vector<vector<int>> map(m, vector<int>(n, 0));
#重新赋值为全0
map.assign(m, vector<int>(n, 0));

#字符串转数值
int i = std::stoi("42");          
long l = std::stol("9999999999");
float f = std::stof("3.14");
double d = std::stod("2.71828");
#数值转字符串
std::string s1 = std::to_string(42);      
std::string s2 = std::to_string(3.14159); 
```
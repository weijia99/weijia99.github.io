---
title: 常用的api
date: '2024-08-27 10:05:41'
updated: '2024-09-28 11:59:37'
description: 'char字符串进行加减法使用ord(c)来获得unicode代码，之后chr来进行编码回去 s = list(s) for i,c in enumerate(s):     dis = min(ord(c)-ord(''a''), ord(''z'')-ord(c)+1)     if dis '
---
char字符串进行加减法

```python
使用ord(c)来获得unicode代码，之后chr来进行编码回去
s = list(s)
for i,c in enumerate(s):
    dis = min(ord(c)-ord('a'), ord('z')-ord(c)+1)
    if dis <= k:
        s[i] = 'a'
        k -= dis
    else:
        s[i]=chr(ord(c)-k)
        break
        return ''.join(s)
```

## 字典
```yaml
rec.get(num,0) #不存在默认返回，与java类似，存在直接使用rec[num]
```

## 优先队列


直接使用get还有put来操作

需要借助list，存放数据

heapq是操作方法函数，然后使用push

```yaml
heapq.heappush(heap,(value,key))
```

移除

也是借助heap来进行heappop，heap[0][0]已经是最小的

默认是最小堆

```yaml
 heap = []
        for key,value in rec.items():
            if len(heap) < k:
                heapq.heappush(heap,(value,key))
            else:
                if value > heap[0][0]:
                    heapq.heappop(heap)
                    heapq.heappush(heap,(value,key))
```



## nonlocal
实现全局的效果，相当于c++的static变量

直接使用self就可以

## 最大最小
使用

```java
return dfs(root,float('-inf'),float('inf'))
```

```java
max(max(row) for row in f)
```



## 自定义排序函数
通过引入functools。cmp_to_key

排序-1，表明最前面

```java
 def order(a,b):
            a=str(a)
            b=str(b)
            if a+b<b+a:
                return -1
            else:
                return 1
        password.sort(key=functools.cmp_to_key(order))
```



# counter实现
```java
 def order(a,b):
            a=str(a)
            b=str(b)
            if a+b<b+a:
                return -1
            else:
                return 1
        password.sort(key=functools.cmp_to_key(order))
```

相当于免去自己使用hashmap了


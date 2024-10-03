---
title: leetcode150
date: 2022-11-19 20:33:58
tags: [leetcode,后序遍历]
---

# leetcode 150

> Evaluate the value of an arithmetic expression in [Reverse Polish Notation](http://en.wikipedia.org/wiki/Reverse_Polish_notation).
>
> Valid operators are `+`, `-`, `*`, and `/`. Each operand may be an integer or another expression.
>
> **Note** that division between two integers should truncate toward zero.
>
> It is guaranteed that the given RPN expression is always valid. That means the expression would always evaluate to a result, and there will not be any division by zero operation.





## 题意

本题是给你了逆波兰表达式，让你自己求，这个结果。



## 思路

我们可以发现，这个就是后缀表达式，使用后缀遍历，然后我们也可以发现，这个树的叶子节点就是数字。

整体思路就是遍历这个字符串，然后发现是数字就push到stack里面，然后发现是符号就pop出来两个数字，对他进行操作。



最后返回栈顶元素



## 代码

```c++
class Solution {
public:
    
    stack<long long> s;
    
    int evalRPN(vector<string>& tokens) {
        // 简单的后缀表达式求和
        set<string> rec;
        rec.insert("+");
        rec.insert("-");
        rec.insert("*");
        rec.insert("/");

        for(auto i:tokens){
            
            if(rec.find(i)!=rec.end()){
                int a=s.top();
                s.pop();
                int b=s.top();
                s.pop();
                // cout<<a<<" "<<b<<endl;
                if(i=="+"){
                    s.push(a+b);
                }else if(i=="-"){
                    s.push(b-a);

                }else if(i=="*"){
                    s.push((long long)a*(long long )b);

                }else{
                    s.push(b/a);

                }
            
            }else{
                s.push(atoi(i.c_str()));
            }
        }
        return s.top();

    }
};
```


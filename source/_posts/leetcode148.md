---
title: leetcode148
date: 2022-11-19 20:13:00
tags: [leetcode,linked list]
---

# leetcode 148

> Given the `head` of a linked list, return *the list after sorting it in **ascending order***.

![](https://fastly.jsdelivr.net/gh/weijia99/blog_image@main/16688601431171668860143064.png)

> **Follow up:** Can you sort the linked list in `O(n logn)` time and `O(1)` memory (i.e. constant space)?



## 题意

把链表进行排序输出，但是只能使用o1的空间，nlogn的时间。

## 思路

使用nlogn的方法来进行排序只有快排还有归并，但是都是用来递归，那空间就是on。所以只能使用迭代的方法来进行排序。



地带也是给予归并的，就是我们手动从下到上，手动进行排序。排序完成一层厚，再次进行下一层来排序。



现在我们进行引进dummy还有cur，dummy使用尾插法，来构建新的完整的链表（这是新一层的）



每一层开始的时候p=q=head，

然后q多走i补来达到下一组的开始



然后引进p还有q，pq是两组的开头，对pq进行循环遍历，次数小于1,2,4，（这是分组的方式）。同事还有一个 o，o是2i的位置（表示新的一组开始进行排序，连接），这个都是用cur来进行尾插法。之后再把head=o，开始进行下一组



完成一层厚，我们让head=dummy-》next。





## 例子

> `[4,3,1,7,8,9,2,11,5,6]`.这个进行排序

```angelscript
step=1: (3->4)->(1->7)->(8->9)->(2->11)->(5->6)
step=2: (1->3->4->7)->(2->8->9->11)->(5->6)
step=4: (1->2->3->4->7->8->9->11)->(5->6)
step=8: (1->2->3->4->5->6->7->8->9->11)
```



我们可以看第一轮i=1，dummy->next表示step1这个完整的链表（3417.。），cur都是尾插法，然后p=4，q=3，o=1，34

结束后就是head=o，p=head=1，q=7，然后接着进行更新





## 代码

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
   /*1.首先求出长度n
   2.第一个for i是求出层数
   3.第二个for是求出分了几组，想领2gei是一组
   4.然后对q走到下一个开头，p是当前的开头
   5.对o走到2i的位置，使他们可以接着循环
   6.之后就是常规的归并方法使用while
   7.一层遍历结束完成后，让cur->next=null,同事更新head为dummy-》nexzt表示新的开头
   */
public:
    ListNode* sortList(ListNode* head) {
        // 使用地带进行排序
        int n=0;
        auto cur=head;
        while(cur){
            n++;
            cur=cur->next;
        }
        
        for(int i=1;i<n;i*=2){
             auto dummy=new ListNode(-1);
            //  dummy指示的是一层的开始
            cur=dummy;
            //第一层循环是层数，从第一个开始,1,2,4
            for(int j=1;j<=n;j+=2*i){
                //第二个循环是组好，然后进行比较
                //  在组好开始进行排序 这是一次组里的比较
               
                // auto cur=dummy;使用cur来进行插入
                auto p=head;//第一组开始
                 auto q=p;//第二组开始
                
                //找到下一个组开始的0号
                for(int k=0;k<i&&q;k++){
                    q=q->next;
                    //下一组的开始
                }
                auto o=q;//下一组的开始标签，使用2i开始
                for(int k=0;k<i&&o;k++){
                    o=o->next;
                }
                int l=0;
                int r=0;
                
                // 两个开始,第一组开始都是1开始比较
                while(p&&q&&l<i&&r<i){
                    if(p->val<q->val){
                        l++;
                     cur->next=p;
                     cur=cur->next;
                     p=p->next;
                    }else{
                        r++;
                        cur->next=q;
                        cur=cur->next;
                        q=q->next;
                        //尾插法
                    }
                }
                while(p&&l<i){
                    l++;
                     cur->next=p;
                     cur=cur->next; 
                     p=p->next;
                }
                while(q&&r<i){
                   r++;
                        cur->next=q;
                        cur=cur->next;
                        q=q->next;
                        //尾插法 
                }
                head=o;//下一次开始
                // cout<<head->val<<" ";
            }
            // 整个结束了，插入到开始
            cur->next=NULL;
            //尾插法，是因为要保持开始的顺序不变
            head=dummy->next;
        }
        return head;
    }
};
```


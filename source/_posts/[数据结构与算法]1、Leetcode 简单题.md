---
title: 1、Leetcode 简单题

date: 2017-05-23 22:37:00

categories:
- 数据结构与算法

tags:
- 算法
- 数据结构

---

## 找出合理的IP地址

![](http://i.imgur.com/Z1YaNvX.png)

另外，生成符合要求的IP地址可以用StringBuilder的insert方法

```
/**
Given a string containing only digits, restore it by returning all possible valid IP address combinations.
For example:
Given"25525511135",
return["255.255.11.135", "255.255.111.35"]. (Order does not matter)
**/

import java.util.*;
 
public class Solution {
     
    private boolean isValid(String s, int i, int j, int k){
         
        if ( k < s.length() - 4 || k > s.length() - 2)
            return false; 
        else if ( isRange(s, 0, i + 1)
                    && isRange(s, i + 1, j + 1)
                    && isRange(s, j + 1, k + 1)
                    && isRange(s, k + 1, s.length()))
            return true;
         
        return false;
    }
     
    private boolean isRange(String s, int st, int end){
         
        int value = Integer.valueOf(s.substring(st, end));
         
        if ( end - st > 1 && s.charAt(st) == '0')
            return false;
        if ( value >= 0 && value <= 255)
            return true;
         
        return false;
    }
     
    private void storeIp(List<String> ipList, String s, int i, int j, int k){
         
        StringBuilder sb = new StringBuilder(s);
         
        sb.insert(i + 1, ".");
        sb.insert(j + 2, ".");
        sb.insert(k + 3, ".");
         
        ipList.add(sb.toString());
    }
     
    public ArrayList<String> restoreIpAddresses(String s) {
         
        ArrayList<String> ipList = new ArrayList<>();
         
        if (s.length() < 4)
            return ipList;
         
        for (int i = 0; i < 3; i++)
            for (int j = i + 1; j < i + 4; j++)
                for (int k = j + 1; k < j + 4; k++){
                     
                    if (isValid(s, i, j, k))
                        storeIp(ipList, s, i, j, k);
                }
         
        return ipList;     
    }
}
```

## 链表相加

http://www.cnblogs.com/wuyuegb2312/p/3183214.html

证明链表第一次相遇的位置在入口处：

设p1在入口的时候，p2位于L处（入口处为0，顺时针增加）
设p1行走x后，p2行走2x后在第一次相遇点相遇，则：
x%R = (L + 2x)%r

则：x = kR，L+2x=k'R =>L + x = (k'-k)R，即L+x为R的倍数，x位于入口处
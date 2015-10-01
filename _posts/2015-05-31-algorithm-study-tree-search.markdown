---
layout: blog
title: "Algorithm study - Tree search"
date: 2015-05-31 15:25:18 +0800
comments: true
categories: blog
---

树搜索策略是算法问题里面的一个大分支，本文旨意在于记录学习树搜索过程中的点滴和领悟。

##广度优先搜索##

广度优先算法是一种横向搜索优先的树搜索算法，算法从树的根结点出发，寻找第一级的子节点，并在这一级的字节点中搜索是否有满足最终条件的节点，若搜索到，则结束，若未搜索到满足退出条件的字节点，则自左向右的搜索第一级的字节点的二级所有二级字节点，如此类推。

搜索过程用到的数据结构是*队列FIFO*

###算法过程描述###

搜索过程可以总结为如下几点：

1. 构造由根结点组成的1元队列
2. 验证队列的第一个节点是否为目标节点，如果是，则返回该节点，若不是则转去步骤3
3. 从队列中删除第一个元素，并寻找该元素的所有子节点，并添加至队列
4. 如果队列为空，那么返回失败，若不为空，则转至步骤2

接下来，通过八数码问题说明如何通过广度优先算法找到最终解。例子截图取材自《算法设计与分析导论》。

###算法实现与例子###

8数码问题也称为九宫格问题，8个数字在九宫格中随机出现，需要在有限步数内到达最终状态    

![algorithm]({{site.url}}/img/algorithm/algorithm-1-02.PNG)

广度优先算法过程    

![algorithm]({{site.url}}/img/algorithm/algorithm-1-01.PNG)

Python实现的广度优先代码

{% highlight python %}
# -*- coding: utf-8 -*-
# 
# Author Enix Yu <enix223@163.com>
# License: MIT
# Description:
#    利用广度优先算法，找到当前节点的子节点，然后顺序遍历字节点，如字节点仍有子节点，则把字节点放在队列后面。
#    广度优先算法寻找8码问题的解，不一定有结果。
#
# How to run:
#    $ python tree.py
#

import random
import math
import Queue
import copy

def shuffle(arr):
    """
    实现对数组的元素位置的随机排列
    """
    for i in range(0,len(arr)):
        j = int(math.floor(random.random()*len(arr)))
        arr[i], arr[j] = arr[j], arr[i]
    return arr

def swap(arr, idx1, idx2):
    ch_solv = copy.copy(arr)
    ch_solv[idx1], ch_solv[idx2] = ch_solv[idx2], ch_solv[idx1]
    return ch_solv

def breath_first_search(target, que):
    """
    广度优先利用队列FIFO实现搜索
    
    eg, target = [1,2,3,4,0,5,6,7,8], init = [8,1,0,3,4,7,2,6,5]

           state = init
                |
          state = que.get() <-------------------------+
                |                                     |
          state == target ?                           |
                |                                     |
                |                                     |
               / \                                    |
            Y /   \ N                                 |
    return state   \                                  | 
                    que.remove(state)                 |
                          |                           |  
                    find the child state              |
                          |                           |
                    add the child state to que        |
                          |                           |
                      que empty?                      |
                          |                           |
                       Y / \ N                        |
                        /   \                         |
                return None  \                        |
                              \-----------------------+
    """
    last_state = []
    while(not que.empty()):
        solution = que.get_nowait()
        if target == solution:
            return solution
        else:
            idx = solution.index(0)  #Find out the blank position
            
            if idx - 3 > -1: #Move the element on top                
                ch_solv = swap(solution, idx, idx - 3)
                if ch_solv not in last_state:               
                    que.put_nowait(ch_solv)                    
                    print 'Before: {}, Move top {}, After: {}'.format(solution, ch_solv[idx], ch_solv)
            if idx + 3 < 9: #Move element bottom
                ch_solv = swap(solution, idx, idx + 3)
                if ch_solv not in last_state:               
                    que.put_nowait(ch_solv)
                    print 'Before: {}, Move bottom {}, After: {}'.format(solution, ch_solv[idx], ch_solv)
            if idx - 1 > -1 and idx % 3 != 0: #Move element on the left hand side
                ch_solv = swap(solution, idx, idx - 1)
                if ch_solv not in last_state:            
                    que.put_nowait(ch_solv)
                    print 'Before: {}, Move left {}, After: {}'.format(solution, ch_solv[idx], ch_solv)
            if idx + 1 < 9  and idx % 2 != 0: #Move element on the right hand side
                ch_solv = swap(solution, idx, idx + 1)
                if ch_solv not in last_state:           
                    que.put_nowait(ch_solv)
                    print 'Before: {}, Move right {}, After: {}'.format(solution, ch_solv[idx], ch_solv)

            last_state.append(solution)                        
            if que.empty():            
                return None #No more solution found in child
                
    return None #No more solution found
    
if __name__ == '__main__':
    """
    主程序入口，通过3个例子，分别实现广度优先，深度优先和爬山算法找到最优解
    """
    """
    Ex.1, 利用广度优先，实现8数码问题的求解    

    最终解: [1,2,3,4,0,5,6,7,8], 0代表空位

    1 2 3
    4   5
    6 7 8
    
    利用shuffle函数将最终解的元素顺序重新安排，生成待求解的状态 s

    可移动的元素位置计算方法：
    
    T: idx - 3 > -1 ? OK : None
    B: idx + 3 < 9  ? OK : None
    L: idx - 1 > -1 and idx mod 3 != 0 ? OK : None
    R: idx + 1 < 9  and idx mod 2 != 0 ? OK : None

    1 2 3    T, B, L, R
    4   5
    6 7 8

    1 2 3    T, B, R
      4 5 
    6 7 8

      1 2    B, R
    3 4 5
    6 7 8                             
    """
    target = [1,2,3,4,0,5,6,7,8]
    #s = copy.copy(target)
    #shuffle(s)
    s = [1,2,3,4,7,5,6,0,8]
    print 'Init sequence: {}'.format(s)
    que = Queue.Queue(100)
    que.put(s)
    try:
        solution = breath_first_search(target, que)
        print 'Find out the solution, {}'.format(solution)
    except:
        print 'Can find the solution in 100 steps.'  
{% endhighlight %}

##深度优先算法##

与广度优先算法不一样，深度优先算法是优先遍历节点的子节点。以纵向的方向搜索目标节点。深度优先使用的是栈的数据结构，先进栈的先出栈。

###算法过程描述###

1. 构造由根结点组成的1元栈
2. 验证栈顶的节点是否为最终解，如果是，则返回该节点；如果不是则转至步骤3
3. 从栈顶删除第一个元素，并寻找该元素的所有子节点，并加入栈中（若存在）。
4. 如果栈为空，则返回五有效解，若不为空，则转至步骤2.

###算法实现与例子###

接下来，通过一个例子来说明深度优先算法，并通过python实现该算法。

假设由一个整数集合，S = {7, 5, 2, 1, 10}, 需要寻找该集合的一个子集，并满足 S'的所有元素的和为9.

以下是深度优先的搜索过程：

![algorithm]({{site.url}}/img/algorithm/algorithm-1-03.PNG)

Python实现深度优先算法

{% highlight python %} 
# -*- coding: utf-8 -*-
# 
# Author Enix Yu <enix223@163.com>
# License: MIT
# Description:
#    利用深度优先算法，找到当前节点的子节点，然后再寻找子节点的子节点，直到没有字节点后，
#    回溯到上一层继续其他节点寻找字节点。
#    本例子利用深度优先算法，计算一个结合的子集元素的和为某个指定的值。本算法的关键在与记录
#    已遍历的树的分支，在回溯至上一层的时候，需要重新定位下一个元素在arr的位置所以i.
#
# How to run:
#    $ python depth_first_search.py
#

_DEBUG = True

def depth_first_search(arr, target):
    solution = [0]
    i = 0
    while len(solution) <> 0:        
        if sum(solution) == target: #Solution match target
            return solution
        elif sum(solution) > target: #Solution sum greater than target
            node = solution.pop()
            i = arr.index(node) + 1
            if i < len(arr):
                solution.append(arr[i])
                i += 1
            else:
                node = solution.pop()
                i = arr.index(node) + 1
        else: #Need to push more element to stack
            if i < len(arr):        
                solution.append(arr[i])
                i += 1
            else:
                break #No more element is available in arr

        if _DEBUG:
            print 'Current solution array: {}, arr index i: {}'.format(solution, i)

    return None                


if __name__ == '__main__':
    s = [7, 5, 1, 2, 10]
    target = 9
    solution = depth_first_search(s, target)
    print 'Solution is {}'.format(solution)
{% endhighlight %}

运行结果为

	Current solution array: [0, 7], arr index i: 1
	Current solution array: [0, 7, 5], arr index i: 2
	Current solution array: [0, 7, 1], arr index i: 3
	Current solution array: [0, 7, 1, 2], arr index i: 4
	Current solution array: [0, 7, 1, 10], arr index i: 5
	Current solution array: [0, 7], arr index i: 3
	Current solution array: [0, 7, 2], arr index i: 4
	Solution is [0, 7, 2]
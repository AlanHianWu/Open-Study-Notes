---
tags:
    - question
    - leetcode
    - hard
    - stacks
    - dataStructures
---
---
###### [LeetCode Link](https://leetcode.com/problems/maximum-frequency-stack/)
---

---
# My Notes
---
Wonder why this is a hard question?
seem easy enough

seems like I need two stacks one for order and the other for frequency, or hash map for frequency?
![[Pasted image 20220930015051.png]]
``` python
class FreqStack:

    def __init__(self):
        
        self.freq = {}

    def push(self, val: int) -> None:
        if val in self.freq:
            self.freq[val] += 1
        else:
            self.freq[val] = 1

    def pop(self) -> int:
        stack = [k for k, v in sorted(self.freq.items(), key=lambda x:x[1])][-1]
        
        self.freq[stack] -= 1
        
        if self.freq[stack] == 0:
            del self.freq[stack]
        
        return stack

# Your FreqStack object will be instantiated and called as such:
# obj = FreqStack()
# obj.push(val)
# param_2 = obj.pop()
```

Was thinking just use a hm for this, but seems like the problem is with same values, so the order of the values can not be maintained in  a HM, so I need a statck to track order.

okay looks like I am runing into optimazation problems now
![[Pasted image 20220930021203.png]]
``` python
class FreqStack:

    def __init__(self):
        self.order = []
        self.freq = {}
        
        

    def push(self, val: int) -> None:
        self.order.append(val)
        if val in self.freq:
            self.freq[val] += 1
        else:
            self.freq[val] = 1

    def pop(self) -> int:
        
        # go through the order and find the most frequent
        
        most_freq = 0
        most_freq_val = 0
        for val in self.order:
            if self.freq[val] >= most_freq:
                most_freq = self.freq[val]
                most_freq_val = val

        # print('freq ', most_freq, most_freq_val, self.order)
        
        # remove the val from the back        
        for i in range(len(self.order) - 1, 0, -1):
            if self.order[i] == most_freq_val:
                self.order = self.order[:i] + self.order[i+1:]
                break
        
        
        self.freq[most_freq_val] -= 1
        if self.freq[most_freq_val] == 0:
            del self.freq[most_freq_val]


        return most_freq_val
        


# Your FreqStack object will be instantiated and called as such:
# obj = FreqStack()
# obj.push(val)
# param_2 = obj.pop()
```

Seems like priority queue is not gonna save me here, yeah got to look into orderd dicts for this, insertion memo is good. [link here](https://www.geeksforgeeks.org/ordereddict-in-python/)

### [[2022-10-01]]
oh shit the stack of stacks that neetcode said worked!!
idea is 

| Freq | order_stack |
| ---- | ---------- |
| 1 | [0,1,2] |
| 2 | [0,1] |
| 3 | [0,] |

use a hashmap to keep track of the freq and order at the same time!
![[Pasted image 20221001022215.png]]
``` python
class FreqStack:

    def __init__(self):
        self.stackOfstacks = {}
        self.freq = {}

    def push(self, val: int) -> None:

        if val in self.freq:
            self.freq[val] += 1
        else:
            self.freq[val] = 1

        f = self.freq[val]

        if f in self.stackOfstacks:
            self.stackOfstacks[f].append(val)
        else:
            self.stackOfstacks[f] = [val]
        # print('added to stack', self.stackOfstacks)
        # print('added to dict ', self.freq)

    def pop(self) -> int:
        m = max(self.stackOfstacks.keys())
        # print(m, self.stackOfstacks)
        res = self.stackOfstacks[m].pop()
        if self.stackOfstacks[m] == []:
            # print('deleting ', m, self.stackOfstacks[m])
            del self.stackOfstacks[m]
        self.freq[res] -= 1
        if self.freq[res] == 0:
            del self.freq[res]

        return res
        
        


# Your FreqStack object will be instantiated and called as such:
# obj = FreqStack()
# obj.push(val)
# param_2 = obj.pop()
```

---
# Question
---
Design a stack-like data structure to push elements to the stack and pop the most frequent element from the stack.

Implement the `FreqStack` class:

-   `FreqStack()` constructs an empty frequency stack.
-   `void push(int val)` pushes an integer `val` onto the top of the stack.
-   `int pop()` removes and returns the most frequent element in the stack.
    -   If there is a tie for the most frequent element, the element closest to the stack's top is removed and returned.

>**Example 1:**
**Input**
["FreqStack", "push", "push", "push", "push", "push", "push", "pop", "pop", "pop", "pop"]
[[], [5], [7], [5], [7], [4], [5], [], [], [], [],]
**Output**
[null, null, null, null, null, null, null, 5, 7, 5, 4]

>**Explanation**
FreqStack freqStack = new FreqStack();
freqStack.push(5); // The stack is [5]
freqStack.push(7); // The stack is [5,7]
freqStack.push(5); // The stack is [5,7,5]
freqStack.push(7); // The stack is [5,7,5,7]
freqStack.push(4); // The stack is [5,7,5,7,4]
freqStack.push(5); // The stack is [5,7,5,7,4,5]
freqStack.pop();   // return 5, as 5 is the most frequent. The stack becomes [5,7,5,7,4].
freqStack.pop();   // return 7, as 5 and 7 is the most frequent, but 7 is closest to the top. The stack becomes [5,7,5,4].
freqStack.pop();   // return 5, as 5 is the most frequent. The stack becomes [5,7,4].
freqStack.pop();   // return 4, as 4, 5 and 7 is the most frequent, but 4 is closest to the top. The stack becomes [5,7].

>**Constraints:**
>-   `0 <= val <= 109`
>-   At most `2 * 104` calls will be made to `push` and `pop`.
>-   It is guaranteed that there will be at least one element in the stack before calling `pop`.

---
# Offical Solution
---
#### Approach 1: Stack of Stacks

**Intuition**
Evidently, we care about the frequency of an element. Let `freq` be a `Map` from xx to the number of occurrences of xx.

Also, we (probably) care about `maxfreq`, the current maximum frequency of any element in the stack. This is clear because we must pop the element with the maximum frequency.

The main question then becomes: among elements with the same (maximum) frequency, how do we know which element is most recent? We can use a stack to query this information: the top of the stack is the most recent.

To this end, let `group` be a map from frequency to a stack of elements with that frequency. We now have all the required components to implement `FreqStack`.

**Algorithm**
Actually, as an implementation level detail, if `x` has frequency `f`, then we'll have `x` in all `group[i] (i <= f)`, not just the top. This is because each `group[i]` will store information related to the `i`th copy of `x`.

Afterwards, our goal is just to maintain `freq`, `group`, and `maxfreq` as described above.

``` python
class FreqStack(object):

    def __init__(self):
        self.freq = collections.Counter()
        self.group = collections.defaultdict(list)
        self.maxfreq = 0

    def push(self, x):
        f = self.freq[x] + 1
        self.freq[x] = f
        if f > self.maxfreq:
            self.maxfreq = f
        self.group[f].append(x)

    def pop(self):
        x = self.group[self.maxfreq].pop()
        self.freq[x] -= 1
        if not self.group[self.maxfreq]:
            self.maxfreq -= 1

        return x
```

**Complexity Analysis**
-   Time Complexity: O(1)O(1) for both `push` and `pop` operations.
-   Space Complexity: O(N)O(N), where `N` is the number of elements in the `FreqStack`.

---
# My Thoughts
---


---
# Time Line
---
### [[2022-09-30]]

### [[2022-10-01]]


---
# Tips & Tricks
---
you need to combine two dataStructures to make a better one


---
# Foot Notes

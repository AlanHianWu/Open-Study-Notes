---
tags:
    - question
    - leetcode
    - python
    - easy
    - bits
---
---
###### [LeetCode Link](https://leetcode.com/problems/add-binary/)
---

---
# My Notes
---

This question is similar to [[Plus One]]

not working close though looking through the offical solution...

![[Pasted image 20220921145921.png]]
``` python
class Solution:
    def addBinary(self, a: str, b: str) -> str:
        
        # padding
        if len(a) > len(b):
            z = '0' * (len(a) - len(b))
            b = z + b
        
        elif len(a) < len(b):
            z = '0' * (len(b) - len(a))
            a = z + a
        
        # adding
        ans = ''
        carry = ['0'] * (len(a)+1)
        for i in range(len(a)-1, -1, -1):
            print(i)
            if a[i] == '0' and b[i] == '0' and carry[i] == '0':
                ans += '0'
            
            elif a[i] == '1' and b[i] == '1':
                carry[i+1] = '1'
                if carry[i] == '1':
                    ans += '1'
                else:
                    ans += '0'
            else:
                ans += '1'
        print('ans ', ans)
```

![[Pasted image 20221006005249.png]]
``` python
class Solution:
    def addBinary(self, a: str, b: str) -> str:
        
        a = a[::-1]
        b = b[::-1]
        
        carry = 0
        
        if len(a) > len(b):
            b += '0' * (len(a) - len(b))
        elif len(b) > len(a):
            a += '0' * (len(b) - len(a))
        
        
        res = ''
        for i in range(len(a)):
            
            if carry == 0:
                if a[i] != b[i]:
                    res += '1'
                elif a[i] == '0':
                    res += '0'
                else:
                    carry = 1
                    res += '0'
            else:
                if a[i] != b[i]:
                    res += '0'
                elif a[i] == '0':
                    res += '1'
                    carry = 0
                else:
                    res += '1'

        if carry == 1:
            res += '1'
                    
        res = res[::-1]
        print(res)
        return res
                
```

---
# Question
---
Given two binary strings `a` and `b`, return _their sum as a binary string_.

>**Example 1:**
**Input:** a = "11", b = "1"
**Output:** "100"

>**Example 2:**
**Input:** a = "1010", b = "1011"
**Output:** "10101"

>**Constraints:**
>-   `1 <= a.length, b.length <= 104`
>-   `a` and `b` consist only of `'0'` or `'1'` characters.
>-   Each string does not contain leading zeros except for the zero itself.

---
# Offical Solution
---
#### Overview
There is a simple way using built-in functions:
-   Convert a and b into integers.
-   Compute the sum.
-   Convert the sum back into binary form.

``` python
class Solution:
    def addBinary(self, a, b) -> str:
        return '{0:b}'.format(int(a, 2) + int(b, 2))
```

The overall algorithm has $\mathcal{O}(N + M)O(N+M)$ time complexity and has two drawbacks which could be used against you during the interview.

> 1 . In Java this approach is limited by the length of the input strings a and b. Once the string is long enough, the result of conversion into integers will not fit into Integer, Long or BigInteger:

-   33 1-bits - and b doesn't fit into Integer.
    
-   65 1-bits - and b doesn't fit into Long.
    
-   [500000001](https://docs.oracle.com/javase/8/docs/api/java/math/BigInteger.html) 1-bits - and b doesn't fit into BigInteger.
    

To fix the issue, one could use standard Bit-by-Bit Computation approach which is suitable for quite long input strings.

> 2 . This method has quite low performance in the case of large input numbers.

One could use Bit Manipulation approach to speed up the solution.  
  

---

#### Approach 1: Bit-by-Bit Computation

**Algorithm**
That's a good old classical algorithm, and there is no conversion from binary string to decimal and back here. Let's consider the numbers bit by bit starting from the lowest one and compute the carry this bit will add.

Start from carry = 0. If number a has 1-bit in this lowest bit, add 1 to the carry. The same for number b: if number b has 1-bit in the lowest bit, add 1 to the carry. At this point the carry for the lowest bit could be equal to (00)_2(00)2​, (01)_2(01)2​, or (10)_2(10)2​.

Now append the lowest bit of the carry to the answer, and move the highest bit of the carry to the next order bit.

Repeat the same steps again, and again, till all bits in a and b are used up. If there is still nonzero carry to add, add it. Now reverse the answer string and the job is done.

**Implementation**
``` python
class Solution:
    def addBinary(self, a, b) -> str:
        n = max(len(a), len(b))
        a, b = a.zfill(n), b.zfill(n)
        
        carry = 0
        answer = []
        for i in range(n - 1, -1, -1):
            if a[i] == '1':
                carry += 1
            if b[i] == '1':
                carry += 1
                
            if carry % 2 == 1:
                answer.append('1')
            else:
                answer.append('0')
            
            carry //= 2
        
        if carry == 1:
            answer.append('1')
        answer.reverse()
        
        return ''.join(answer)
```

**Complexity Analysis**
-   Time complexity: $\mathcal{O}(\max(N, M))O(max(N,M))$, where NN and MM are lengths of the input strings a and b.

-   Space complexity: $\mathcal{O}(\max(N, M))O(max(N,M))$ to keep the answer.  

---

#### Approach 2: Bit Manipulation

**Intuition**

Here the input is more adapted to push towards Approach 1, but there is popular Facebook variation of this problem when interviewer provides you two numbers and asks to sum them up without using addition operation.

> No addition? OK, bit manipulation then.

How to start? There is an interview tip for bit manipulation problems: if you don't know how to start, start from computing XOR for your input data. Strangely, that helps to go out for quite a lot of problems, [Single Number II](https://leetcode.com/articles/single-number-ii/), [Single Number III](https://leetcode.com/articles/single-number-iii/), [Maximum XOR of Two Numbers in an Array](https://leetcode.com/articles/maximum-xor-of-two-numbers-in-an-array/), [Repeated DNA Sequences](https://leetcode.com/articles/repeated-dna-sequences/), [Maximum Product of Word Lengths](https://leetcode.com/articles/maximum-product-of-word-lengths/), etc.

Here XOR is a key as well, because it's a sum of two binaries without taking carry into account.

![fig](https://leetcode.com/problems/add-binary/Figures/67/xor4.png)

To find current carry is quite easy as well, it's AND of two input numbers, shifted one bit to the left.

![fig](https://leetcode.com/problems/add-binary/Figures/67/carry2.png)

Now the problem is reduced: one has to find the sum of answer without carry and carry. It's the same problem - to sum two numbers, and hence one could solve it in a loop with the condition statement "while carry is not equal to zero".

**Algorithm**

-   Convert a and b into integers x and y, x will be used to keep an answer, and y for the carry.
    
-   While carry is nonzero: `y != 0`:
    
    -   Current answer without carry is XOR of x and y: `answer = x^y`.
        
    -   Current carry is left-shifted AND of x and y: `carry = (x & y) << 1`.
        
    -   Job is done, prepare the next loop: `x = answer`, `y = carry`.
        
-   Return x in the binary form.
    

**Implementation**

``` python
class Solution:
    def addBinary(self, a, b) -> str:
        x, y = int(a, 2), int(b, 2)
        while y:
            answer = x ^ y
            carry = (x & y) << 1
            x, y = answer, carry
        return bin(x)[2:]
```

This solution could be written as 4-liner in Python

``` python
class Solution:
    def addBinary(self, a, b) -> str:
        x, y = int(a, 2), int(b, 2)
        while y:
            x, y = x ^ y, (x & y) << 1
        return bin(x)[2:]
```

**Performance Discussion**

Here we deal with input numbers which are greater than $2^{100}2100$. That forces to use slow [BigInteger](https://docs.oracle.com/javase/8/docs/api/java/math/BigInteger.html) in Java, and hence the performance gain will be present for the Python solution only. Provided here Java solution could make its best with Integers or Longs, but not with BigIntegers.

**Complexity Analysis**

-   Time complexity : $\mathcal{O}(N + M)O(N+M)$, where NN and MM are lengths of the input strings a and b.
    
-   Space complexity : $\mathcal{O}(\max(N, M))O(max(N,M))$ to keep the answer.

---
# Time Line
---
### [[2022-09-21]]
ditching this for now tbh. revisit later

### [[2022-10-02]] 
this is later, yep still to confusing

### [[2022-10-06]]
not confusing any more

---
# Tips & Tricks
---
the key to this problem is figuring out how to carry a value over.

---
# Foot Notes

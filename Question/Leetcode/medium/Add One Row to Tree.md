---
tags:
    - medium
    - binary_tree
---
---
###### [LeetCode Link](https://leetcode.com/problems/add-one-row-to-tree/)
---

# My Notes
seems to just be add node at depth x, but I'll have to do it for left and right.

### [[2022-10-12]]
seems like a realy complex problem, gonna need to first find the node at depth x and then add the nodes, and do it across the entire tree

seems like a BFS appoarch is best.

just working on it rn, have BFS done, and depth counter is working just fine, now the only problem is determining where to add the child, and adding None I suppose.
``` python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def addOneRow(self, root: Optional[TreeNode], val: int, depth: int) -> Optional[TreeNode]:
        # BFS
        queue = [(None, None, root, 0)]
        
        while queue:
            parent, direction, node, Depth = queue.pop(-1)
            if node:
                print('At ', node.val, Depth)
            
            if Depth == depth:
                newNode = TreeNode(val)
                newNode.left = parent.left
                newNode.right = parent.right
                
                # parent direction?
                if direction == 'L':
                    parent.left = newNode
                else:
                    parent.right = newNode
            else:
                nodeL = (node, 'L', node.left, Depth+1)
                nodeR = (node, 'R', node.right, Depth+1)
                
                queue.append(nodeL)
                queue.append(nodeR)
            return root
```

seems like it was stated here
>The adding rule is:
>-   Given the integer `depth`, for each not null tree node `cur` at the depth `depth - 1`, create two tree nodes with value `val` as `cur`'s left subtree root and right subtree root.
>-   `cur`'s original left subtree should be the left subtree of the new left subtree root.
>-   `cur`'s original right subtree should be the right subtree of the new right subtree root.
>-   If `depth == 1` that means there is no depth `depth - 1` at all, then create a tree node with value `val` as the new root of the whole original tree, and the original tree is the new root's left subtree.
>- 

forgot to add visited again.
![[Pasted image 20221012110045.png]]
seems like the adding is not working right. so it seems that the first depth set to 0 is not right changing it to 0
![[Pasted image 20221012110217.png]]
seems like there is dups getting added. 
setting the oppsite child to be added to None seems to have fixed that
![[Pasted image 20221012110432.png]]
hmmm...??

seems like the starting parms for adding left and right is broken, and the logic for adding is messed up,

maybe I got to check is the nodes have any children, to determine where to add the nodes? wait you are always gonna add two?

looking better now
![[Pasted image 20221012112516.png]]

might be a travel error, as in the order in which I am add into the queue is srewed

![[Pasted image 20221012112851.png]]
alright time for edgecases
``` python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def addOneRow(self, root: Optional[TreeNode], val: int, depth: int) -> Optional[TreeNode]:

        # BFS
        queue = [(root, root, 1)]
        visited = set()
        
        while queue:
            parent, node, Depth = queue.pop(-1)
            if node:
                # print('At ', node.val, Depth)
            
                if Depth == depth and node not in visited:
                    newNodeL = TreeNode(val)
                    newNodeR = TreeNode(val)
                    newNodeL.left = parent.left
                    newNodeR.right = parent.right
                    
                    parent.left = newNodeL
                    parent.right = newNodeR

                    queue.append((newNodeL, parent.left, Depth+1))
                    queue.append((newNodeR, parent.right, Depth+1))

                else:

                    nodeL = (node, node.left, Depth+1)
                    nodeR = (node, node.right, Depth+1)

                    visited.add(node)
                    queue.append(nodeL)
                    queue.append(nodeR)    
        return root
            
```

first edge case is when the depth is give +1 of the last depth of th node tree.
so in the description
> -   If `depth == 1` that means there is no depth `depth - 1` at all, then create a tree node with value `val` as the new root of the whole original tree, and the original tree is the new root's left subtree.

keeps adding at the second last node for some reason
``` python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def addOneRow(self, root: Optional[TreeNode], val: int, depth: int) -> Optional[TreeNode]:

        # BFS
        queue = [(root, root, 1)]
        visited = set()
        
        while queue:
            parent, node, Depth = queue.pop(-1)
            if node:
                print('At ', node.val, Depth)
                
                end = (Depth+1 == depth) and (node.left == None and node.right == None)
                
                if (Depth == depth and node not in visited) or end:
                    
                    newNodeL = TreeNode(val)
                    newNodeR = TreeNode(val)
                    newNodeL.left = parent.left
                    newNodeR.right = parent.right

                    parent.left = newNodeL
                    parent.right = newNodeR
    
                    if Depth+1 == depth:
                        Depth += 1
                    else:
                        queue.append((newNodeL, parent.left, Depth+1))
                        queue.append((newNodeR, parent.right, Depth+1))

                else:

                    nodeL = (node, node.left, Depth+1)
                    nodeR = (node, node.right, Depth+1)

                    visited.add(node)
                    queue.append(nodeL)
                    queue.append(nodeR)

                
        return root
            
            
```
oh yeah I am just check for the next node and not switch over to the next

![[Pasted image 20221012115120.png]]
nearly there

just had to move to the next node for the last problem, now what is the issue here?
so
![[Pasted image 20221012115302.png]]
if the depth is 1 you add it on top is not happening...
so 
``` python
if depth == 1:
	newHead = TreeNode(val)
	newHead.left = root
	newHead.right = None
	return newHead
```

cool did it
![[Pasted image 20221012115555.png]]
very slow uhh.

---
# Question
---
Given the `root` of a binary tree and two integers `val` and `depth`, add a row of nodes with value `val` at the given depth `depth`.

Note that the `root` node is at depth `1`.

The adding rule is:

-   Given the integer `depth`, for each not null tree node `cur` at the depth `depth - 1`, create two tree nodes with value `val` as `cur`'s left subtree root and right subtree root.
-   `cur`'s original left subtree should be the left subtree of the new left subtree root.
-   `cur`'s original right subtree should be the right subtree of the new right subtree root.
-   If `depth == 1` that means there is no depth `depth - 1` at all, then create a tree node with value `val` as the new root of the whole original tree, and the original tree is the new root's left subtree.

>**Example 1:**
>![](https://assets.leetcode.com/uploads/2021/03/15/addrow-tree.jpg)
**Input:** root = [4,2,6,3,1,5], val = 1, depth = 2
**Output:** [4,1,1,2,null,null,6,3,1,5]

>**Example 2:**
>![](https://assets.leetcode.com/uploads/2021/03/11/add2-tree.jpg)
**Input:** root = [4,2,null,3,1], val = 1, depth = 3
**Output:** [4,2,null,1,1,3,null,null,1]

>**Constraints:**
>-   The number of nodes in the tree is in the range `[1, 104]`.
>-   The depth of the tree is in the range `[1, 104]`.
>-   `-100 <= Node.val <= 100`
>-   `-105 <= val <= 105`
>-   `1 <= depth <= the depth of tree + 1`

---
# Offical Solution
---

#### Approach #1 Using Recursion(DFS) [Accepted]

If the given depth dd happens to be equal to 1, we can directly put the whole current tree as a left child of the newly added node. Otherwise, we need to put the new node at appropriate levels.

To do so, we make use of a recursive function `insert(val,node,depth,n)`. Here, valval refers to the value of the new node to be inserted, depthdepth refers to the depth of the node currently considered, nodenode refers to the node calling the current function for its child subtrees and nn refers to the height at which the new node needs to be inserted.

For inserting the new node at appropriate level, we can start by making a call to `insert` with the root node and 1 as the current level. Inside every such call, we check if we've reached one level prior to the level where the new node needs to be inserted.

From this level, we can store the roots of the left and right subtrees of the current node temporarily, and insert the new node as the new left and right subchild of the current node, with the temporarily stored left and right subtrees as the left and right subtrees of the newly inserted left or right subchildren appropriately.

But, if we haven't reached the destined level, we keep on continuing the recursive calling process with the left and right children of the current node respectively. At every such call, we also incrmenet the depth of the current level to reflect the depth change appropriately.

``` java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    public TreeNode addOneRow(TreeNode t, int v, int d) {
        if (d == 1) {
            TreeNode n = new TreeNode(v);
            n.left = t;
            return n;
        }
        insert(v, t, 1, d);
        return t;
    }

    public void insert(int val, TreeNode node, int depth, int n) {
        if (node == null)
            return;
        if (depth == n - 1) {
            TreeNode t = node.left;
            node.left = new TreeNode(val);
            node.left.left = t;
            t = node.right;
            node.right = new TreeNode(val);
            node.right.right = t;
        } else {
            insert(val, node.left, depth + 1, n);
            insert(val, node.right, depth + 1, n);
        }
    }
}

```

**Complexity Analysis**
-   Time complexity : $O(n)O(n)$. A total of nn nodes of the given tree will be considered.

-   Space complexity : $O(n)O(n)$. The depth of the recursion tree can go upto nn in the worst case(skewed tree).

---

#### Approach #2 Using stack(DFS) [Accepted]

**Algorithm**
We can do the same task as discussed in the last approach by making use of a stackstack as well. But, we need to make use of a new data structure, NodeNode here, to keep a track of the depth of the current node along with its value.

We start by pushing the root NodeNode onto the stackstack. Then, at every step we do as follows:

-   Pop an element from the stackstack.
-   For every Node popped, check if its depth corresponds to one prior to the depth at which the new node needs to be inserted.
-   If yes, insert the new nodes appropriately as in the last approach.
-   If no, we push both the left and the right child Node(value+depth) of the current node onto the stackstack.
-   Continue the popping and pushing process till the stackstack becomes empty.

``` java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    class Node{
        Node(TreeNode n,int d){
            node=n;
            depth=d;
        }
        TreeNode node;
        int depth;
    }
    public TreeNode addOneRow(TreeNode t, int v, int d) {
        if (d == 1) {
            TreeNode n = new TreeNode(v);
            n.left = t;
            return n;
        } 
        Stack<Node> stack=new Stack<>();
        stack.push(new Node(t,1));
        while(!stack.isEmpty())
        {
            Node n=stack.pop();
            if(n.node==null)
                continue;
            if (n.depth == d - 1 ) {
                TreeNode temp = n.node.left;
                n.node.left = new TreeNode(v);
                n.node.left.left = temp;
                temp = n.node.right;
                n.node.right = new TreeNode(v);
                n.node.right.right = temp;
                
            } else{
                stack.push(new Node(n.node.left, n.depth + 1));
                stack.push(new Node(n.node.right, n.depth + 1));
            }
        }
        return t;
    }
}

```

**Complexity Analysis**
-   Time complexity : $O(n)O(n)$. A total of nn nodes of the given tree will be considered.
-   Space complexity : $O(n)O(n)$. The depth of the stackstack can go upto nn in the worst case(skewed tree).

---
#### Approach #3 Using queue(BFS) [Accepted]
**Algorithm**
The idea of traversal in the last approach is similar to Depth First Search. In that case, we need to traverse through all the nodes of the given tree in the order of branches. Firstly we explored one branch to as much depth as possible and then continued with the other ones.

If, instead, we go for Breadth First Search, along with keeping track of the depth of the nodes being considered at any moment during the Breadth First Search, we can stop the search process as soon as all the nodes at the depth d - 1d−1 have been considered once.

To implement this BFS, we make use of a queuequeue. We start off by pushing the root node of the given tree at the back of the queuequeue and with the depth of the current level set as 1. Then, at every step, we do the following:

-   Remove an element from the front of the queuequeue and add all its children to the back of another temporary queue, temptemp.
-   Keep on adding the elements to the back of the temptemp till queuequeue becomes empty. (Once queuequeue becomes empty, it indicates that all the nodes at the current level have been considered and now temptemp contains all the nodes lying at the next level).
-   Reinitialize queuequeue with its value as temptemp. Update the current value of the depthdepth to reflect the level of nodes currently being considered.
-   Repeat the process till we reach the depth d - 1d−1.
-   On hitting this depth level(d-1d−1), add the new nodes appropriately to all the nodes in the queuequeue currently, as done in the previous approaches.

``` java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
public class Solution {
    public TreeNode addOneRow(TreeNode t, int v, int d) {
        if (d == 1) {
            TreeNode n = new TreeNode(v);
            n.left = t;
            return n;
        }
        Queue < TreeNode > queue = new LinkedList < > ();
        queue.add(t);
        int depth = 1;
        while (depth < d - 1) {
            Queue < TreeNode > temp = new LinkedList < > ();
            while (!queue.isEmpty()) {
                TreeNode node = queue.remove();
                if (node.left != null) temp.add(node.left);
                if (node.right != null) temp.add(node.right);
            }
            queue = temp;
            depth++;
        }
        while (!queue.isEmpty()) {
            TreeNode node = queue.remove();
            TreeNode temp = node.left;
            node.left = new TreeNode(v);
            node.left.left = temp;
            temp = node.right;
            node.right = new TreeNode(v);
            node.right.right = temp;
        }
        return t;
    }
}
```

**Complexity Analysis**
-   Time complexity : O(n)O(n). A total of nn nodes of the given tree will be considered in the worst case.
    
-   Space complexity : O(x)O(x). The size of the queuequeue or temptemp queue can grow upto xx only. Here, xx refers to the number of maximum number of nodes at any level in the given tree.

---
# My Thoughts
---
so there is a recurive solution and stuff.

---
# Time Line
---
### [[2022-10-06]]

### [[2022-10-12]]


---
# Tips & Tricks
---

---
# Foot Notes




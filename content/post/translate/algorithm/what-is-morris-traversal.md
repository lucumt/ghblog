---
title: "[译]二叉树的莫里斯遍历"
date: 2023-08-18T11:18:47+08:00
lastmod: 2023-08-18T11:18:47+08:00
draft: false
keywords: ["二叉树","遍历","莫里斯"]
description: "简要介绍如何使用莫里斯算法在不适用递归和栈的情形下对二叉树进行遍历"
tags: ["二叉树"]
categories: ["翻译","算法"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: true
borderImage: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

本文翻译自[**What is Morris traversal**](https://www.educative.io/answers/what-is-morris-traversal)

<!--more-->

**莫里斯(中序)遍历**是一种对二叉树不使用递归或堆栈进行便利的算法，在此种遍历中，将创建链接作为后继节点并利用它们打印节点，遍历完成后将更改恢复为原始二叉树。

## 算法

* 将`root`节点赋值给`curr`变量节点
* 当`curr`不为空时，检查`curr`是否有左子节点
* 若`curr`没有左子节点，打印`curr`节点并将其指向`curr`右边的节点
* 若`curr`有左子节点，将`curr`指向其左子树中的最右节点的右子节点
* 将`curr`指向此左节点

## 示例

以下图所示的二叉树为例演示如何使用莫里斯遍历

 ![初始二叉树](/blog_img/translate/what-is-morris-traversal/binary_tree.svg "初始二叉树") 

根节点的值为4，将其初始化赋值给`curr`,4有左子节点，因此其成为其左子树中最右边节点的右子节点(中序遍历中节点4的前一个直接节点)，最终节点4成为节点3的右子节点同时`curr`的值被设置为2。

 ![前两次遍历](/blog_img/translate/what-is-morris-traversal/1st_and_2nd_iterations.svg "前两次遍历") 

现在`curr`的值为2，拥有左右子节点且右节点被连接到root，因此我们将`curr`设置为节点1，`curr`现在没有子节点，其右节点将会指向节点2。

 ![第3次和第4次遍历](/blog_img/translate/what-is-morris-traversal/3rd_and_4th_iterations.svg "第3次和第4次遍历") 

节点1被输出是由于其没有左子节点并且`curr`指向了节点2，在第1轮的迭代中其已经成为了节点1的右子节点。在接下来的迭代中，节点2同时具有左右子节点。但是，循环的双重条件使得它在到达自身时就停止，这意味着其左子树已经遍历完毕，因此其输出自身值并继续处理右子节点3。输出节点3之后，`curr`指向节点3并执行同节点2类似的操作。其可发现左子树已经被遍历并继续对节点4进行相同的操作，之后二叉树其余的部分仿照类似的步骤进行遍历。

## 代码

下面用不同的语言实现了上述算法

 {{% codetabs %}}   

```c::C++版本
#include <iostream>
using namespace std; 

struct Node { 
	int data; 
	struct Node* left_node; 
	struct Node* right_node; 
}; 

void Morris(struct Node* root) 
{ 
	struct Node *curr, *prev; 

	if (root == NULL) 
		return; 

	curr = root; 

	while (curr != NULL) { 

		if (curr->left_node == NULL) { 
			cout << curr->data << endl; 
			curr = curr->right_node; 
		} 

		else { 

			/* Find the previous (prev) of curr */
			prev = curr->left_node; 
			while (prev->right_node != NULL && prev->right_node != curr) 
				prev = prev->right_node; 

			/* Make curr as the right child of its
			previous */
			if (prev->right_node == NULL) { 
				prev->right_node = curr; 
				curr = curr->left_node; 
			} 

			/* fix the right child of previous */
			else { 
				prev->right_node = NULL; 
				cout << curr->data << endl; 
				curr = curr->right_node; 
			} 
		} 
	} 
} 

struct Node* add_node(int data) 
{ 
	struct Node* node = new Node; 
	node->data = data; 
	node->left_node = NULL; 
	node->right_node = NULL; 

	return (node); 
} 

int main() 
{ 

	struct Node* root = add_node(4); 
	root->left_node = add_node(2); 
	root->right_node = add_node(5); 
	root->left_node->left_node = add_node(1); 
	root->left_node->right_node = add_node(3); 
	Morris(root); 
	return 0; 
} 
```

```python::Python版本
class Node: 
	def __init__(self, data): 
		self.data = data 
		self.left_node = None
		self.right_node = None

def Morris(root): 
	# Set current to root of binary tree 
	curr = root 
	
	while(curr is not None): 
		
		if curr.left_node is None: 
			print curr.data, 
			curr = curr.right_node 
		else: 
			# Find the previous (prev) of curr 
			prev = curr.left_node 
			while(prev.right_node is not None and prev.right_node != curr): 
				prev = prev.right_node 

			# Make curr as right child of its prev 
			if(prev.right_node is None): 
				prev.right_node = curr 
				curr = curr.left_node 
				
			# fix the right child of prev
			else: 
				prev.right_node = None
				print curr.data, 
				curr = curr.right_node 
			

root = Node(4) 
root.left_node = Node(2) 
root.right_node = Node(5) 
root.left_node.left_node = Node(1) 
root.left_node.right_node = Node(3) 

Morris(root) 
```

```java::Java版本
class Node { 
	int data; 
	Node left_node, right_node; 

	Node(int item) { 
		data = item; 
		left_node =  null;
		right_node = null; 
	} 
} 

class Tree { 
	Node root; 

	void Morris(Node root) { 
		Node curr, prev; 

		if (root == null) {
		   return; 
		}

		curr = root; 
		while (curr != null) { 
			if (curr.left_node == null) { 
				System.out.print(curr.data + " "); 
				curr = curr.right_node; 
			} else { 
				/* Find the previous (prev) of curr */
				prev = curr.left_node; 
				while (prev.right_node != null && prev.right_node != curr) {
					prev = prev.right_node; 
				}

				/* Make curr as right child of its prev */
				if (prev.right_node == null) { 
					prev.right_node = curr; 
					curr = curr.left_node; 
				} 

				/* fix the right child of prev*/

				else { 
					prev.right_node = null; 
					System.out.print(curr.data + " "); 
					curr = curr.right_node; 
				} 

			} 

		}
	} 

	public static void main(String args[]) { 
		
		Tree tree = new Tree(); 
		tree.root = new Node(4); 
		tree.root.left_node = new Node(2); 
		tree.root.right_node = new Node(5); 
		tree.root.left_node.left_node = new Node(1); 
		tree.root.left_node.right_node = new Node(3); 

		tree.Morris(tree.root); 
	} 
} 
```

 {{% /codetabs %}}   

注：由于尚未掌握此算法，所以只是生搬硬套的翻译，质量不高请见谅！
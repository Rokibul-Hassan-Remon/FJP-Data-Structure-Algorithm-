## 📝 Problem Statement:

You are given the root node of a **Binary Search Tree (BST)** and an integer value `key`.  
Your task is to **delete the node with the given key** from the BST while maintaining the **BST property**:

> For every node,  
> `node.left.value < node.value < node.right.value`

If the `key` is **not present** in the BST, the tree should remain **unchanged**.

---

## ⚙️ Rules of Deletion
## 📥 Input Format

- The first line contains an integer `n` — number of nodes in the BST.
    
- The second line contains `n` space-separated integers — the **preorder traversal** of the BST.
    
- The third line contains the integer `key` — the node value to delete.
    

**Constraints:**
```css
	1 ≤ n ≤ 10^5
	-10^9 ≤ node.value, key ≤ 10^9
```

---

## 📤 Output Format

- Print the **inorder traversal** of the BST **after deletion**.
    

---

## 🧮 Example

### Input:
```css
	7
	50 30 20 40 70 60 80
	50
```

Output:
```css
20 30 40 60 70 80

```


## 🧠 Explanation:

Initial BST:
```css
        50
       /  \
     30    70
    / \    / \
   20 40  60 80

```

After deleting `50`:

- The node `50` has **two children**.
    
- Replace `50` with its **in-order successor**, i.e., `60`.
    
- Then remove `60` from right subtree.


Resulting BST:
```css
        60
       /  \
     30    70
    / \      \
   20 40      80

```
In-order Traversal → `20 30 40 60 70 80`

```css
        40
       /  \
     30    70
    /      / \
   20     60 80
```
In-order Traversal → `20 30 40 60 70 80`

---
title: 算法【腾讯50题Swift版】上篇
date: 2023-09-18 21:44:18
tags:
- 算法
- 力扣
---

##### 摘要

当前想在力扣刷算法题，提交语言是没有OC可供选择的，那作为iOS开发者就只能转而使用 Swift 去提交。好在作为程序员掌握了一门语言，再去学习一门其他的语言还是比较容易的。要学习 Swift 去看下 Swift 的官方文档，看个一两周，掌握了语言基础再去刷题，还是比较轻松的。本文则是记录我在力扣用 Swift 刷题的一些题解，本篇记录腾讯精选50题，我这里根据题目的难易度，按照简单、中等、困难的顺序排列。

---

##### [9. 回文数](https://leetcode.cn/problems/palindrome-number/description/)

> 简单题，一个数是回文数，那么首尾对应的数字都相同，转成字符串逐一比对首尾字符即可。

```Swift
class Solution {
    func isPalindrome(_ x: Int) -> Bool {
        guard x >= 0 else { return false }
        var num = String(x)
        while num.count > 1 {
            guard num.removeFirst() == num.removeLast() else { return false }
        }
        return true
    }
}
```

##### [14. 最长公共前缀](https://leetcode.cn/problems/longest-common-prefix/description/)

> 简单题，取出首元素，遍历后面的每个元素，不断更新公共前缀

```Swift
class Solution {
    func longestCommonPrefix(_ strs: [String]) -> String {
        guard strs.count > 0 else { return "" }
        var ans = Array(strs[0])
        for i in 1..<strs.count {
            let chars = Array(strs[i])
            var count = 0
            for j in 0..<min(chars.count, ans.count) {
                if ans[j] != chars[j] {
                    break
                }
                count += 1
            }
            ans = Array(ans[0..<count])
            if ans.isEmpty {
                return ""
            }
        }
        return String(ans)
    }
}
```

##### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/description/)

> 简单题，本题关键是理解，最内层左括号紧接着的下一个必须是与之相符的右括号，否则就无法关闭

```Swift
class Solution {
    func isValid(_ s: String) -> Bool {
        let map: [Character: Character] = ["{": "}", "[": "]", "(": ")"]
        var stack = [Character]()
        for char in s {
            if map[char] != nil {
                stack.append(char)
            } else {
                guard let top = stack.popLast(), map[top] == char else {
                    return false
                }
            }
        }
        return stack.isEmpty
    }
}
```

##### [21. 合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/description/)

> 简单题，递归处理

```Swift
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public var val: Int
 *     public var next: ListNode?
 *     public init() { self.val = 0; self.next = nil; }
 *     public init(_ val: Int) { self.val = val; self.next = nil; }
 *     public init(_ val: Int, _ next: ListNode?) { self.val = val; self.next = next; }
 * }
 */
class Solution {
    func mergeTwoLists(_ list1: ListNode?, _ list2: ListNode?) -> ListNode? {
        guard let l1 = list1 else {return list2}
        guard let l2 = list2 else {return list1}
        if l1.val < l2.val {
            l1.next = mergeTwoLists(l1.next, l2)
            return l1
        } else {
            l2.next = mergeTwoLists(l1, l2.next)
            return l2
        }
    }
}
```

##### [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/description/)

> 简单题，用集合处理，注意 nums 是 inout 参数，要将其修改为操作后的数组

```Swift
class Solution {
    func removeDuplicates(_ nums: inout [Int]) -> Int {
        var aset: Set<Int> = Set()
        for item in nums {
            if aset.contains(item) {
                continue
            }
            aset.insert(item)
        }
        nums = aset.sorted()
        return nums.count
    }
}
```

##### [70. 爬楼梯](https://leetcode.cn/problems/climbing-stairs/description/)

> 简单题，典型的动态规划，爬到第n阶等于，爬到第n-1阶和n-2阶方法之和，用2个变量交替存值，循环即可

```Swift
class Solution {
    func climbStairs(_ n: Int) -> Int {
        if n <= 2 {
            return n
        }
        var f1 = 1, f2 = 2, sum = 0
        for _ in 3...n {
            sum = f1 + f2
            f1 = f2
            f2 = sum
        }
        return sum
    }
}
```

##### [88. 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/description/)

> 简单题，合并后再排序即可

```Swift
class Solution {
    func merge(_ nums1: inout [Int], _ m: Int, _ nums2: [Int], _ n: Int) {
        nums1 = nums1[0..<m] + nums2
        nums1.sort()
    }
}
```

##### [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/description/)

> 简单题，递归即可，注意传入depth

```Swift
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     public var val: Int
 *     public var left: TreeNode?
 *     public var right: TreeNode?
 *     public init() { self.val = 0; self.left = nil; self.right = nil; }
 *     public init(_ val: Int) { self.val = val; self.left = nil; self.right = nil; }
 *     public init(_ val: Int, _ left: TreeNode?, _ right: TreeNode?) {
 *         self.val = val
 *         self.left = left
 *         self.right = right
 *     }
 * }
 */
class Solution {
    func maxDepth(_ root: TreeNode?) -> Int {
        var maxDepth = -1
        func dfs(_ node: TreeNode?, _ depth: Int) {
            if node == nil {
                maxDepth = maxDepth > depth ? maxDepth : depth
                return
            }
            dfs(node?.left, depth+1)
            dfs(node?.right, depth+1)
        }
        dfs(root, 0)
        return maxDepth
    }
}
```

##### [121. 买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/description/)

> 简单题，动态规划，不断计算前n天的最小价格 和 当天卖出与之前某天卖出的最大值

```Swift
class Solution {
    func maxProfit(_ prices: [Int]) -> Int {
        var minPrice = Int.max
        var maxResult = 0

        for price in prices {
            minPrice = min(price, minPrice)
            maxResult = max(maxResult, price - minPrice)
        }
        return maxResult
    }
}
```

##### [136. 只出现一次的数字](https://leetcode.cn/problems/single-number/description/)

> 简单题，用异或来做，异或：同为0异为1。两个相同的数都会被抵消为0，0异或任何数，等于任何数。

```Swift
class Solution {
    func singleNumber(_ nums: [Int]) -> Int {
        return nums.reduce(0) {$0 ^ $1}
    }
}
```

##### [141. 环形链表](https://leetcode.cn/problems/linked-list-cycle/description/)

> 简单题，快慢指针，如果是环形链表快慢指针必定会相遇

```Swift
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public var val: Int
 *     public var next: ListNode?
 *     public init(_ val: Int) {
 *         self.val = val
 *         self.next = nil
 *     }
 * }
 */

class Solution {
    func hasCycle(_ head: ListNode?) -> Bool {
        var node1: ListNode? = head
        var node2: ListNode? = head
        while node1?.next != nil {
            node1 = node1!.next?.next
            node2 = node2?.next
            if node1 === node2 {
                return true
            }
        }
        return false
    }
}
```

##### [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/description/)

> 简单题，两指针各3段走完，路程一样必相遇于交点

```Swift
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public var val: Int
 *     public var next: ListNode?
 *     public init(_ val: Int) {
 *         self.val = val
 *         self.next = nil
 *     }
 * }
 */

class Solution {
    func getIntersectionNode(_ headA: ListNode?, _ headB: ListNode?) -> ListNode? {
        var l1 = headA
        var l2 = headB
        while l1 !== l2 {
            l1 = l1 == nil ? headB : l1!.next
            l2 = l2 == nil ? headA : l2!.next
        }
        return l1
    }
}
```

##### [169. 多数元素](https://leetcode.cn/problems/majority-element/description/)

> 简单题，用字典记录出现次数

```Swift
class Solution {
    func majorityElement(_ nums: [Int]) -> Int {
        var dic: [Int: Int] = [:]
        for item in nums {
            dic[item] = dic[item] == nil ? 1 : dic[item]! + 1
        }
        let m = nums.count >> 1
        for (key, value) in dic {
            if value > m {
                return key
            }
        }
        return 0
    }
}
```

##### [206. 反转链表](https://leetcode.cn/problems/reverse-linked-list/description/)

> 简单题，用temp记录下一指针，依次交换

```Swift
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public var val: Int
 *     public var next: ListNode?
 *     public init() { self.val = 0; self.next = nil; }
 *     public init(_ val: Int) { self.val = val; self.next = nil; }
 *     public init(_ val: Int, _ next: ListNode?) { self.val = val; self.next = next; }
 * }
 */
class Solution {
    func reverseList(_ head: ListNode?) -> ListNode? {
        if head == nil || head?.next == nil {
            return head
        }
        var pre: ListNode? = nil
        var cur: ListNode? = head
        var temp: ListNode? = nil
        while cur != nil {
            temp = cur?.next
            cur?.next = pre
            pre = cur
            cur = temp
        }
        return pre
    }
}
```

##### [217. 存在重复元素](https://leetcode.cn/problems/contains-duplicate/description/)

> 简单题，用集合或排序数组后遍历来做都可以，比较简单

```Swift
class Solution {
    func containsDuplicate(_ nums: [Int]) -> Bool {
        var set = Set<Int>()
        for num in nums {
            if set.contains(num) {
                return true
            }
            set.insert(num)
        }
        return false
    }
}
```

##### [231. 2 的幂](https://leetcode.cn/problems/power-of-two/description/)

> 简单题，2种方法，利用持续除2为1或利用位运算，n & (n-1) 必定为0都可以

```Swift
class Solution {
    func isPowerOfTwo(_ n: Int) -> Bool {
        var m = n
        while m > 0 && m % 2 == 0 {
            m = m / 2
        }
        return m == 1
    }
}
```

```Swift
class Solution {
    func isPowerOfTwo(_ n: Int) -> Bool {
        guard n>0 else { return false }
        return n & (n-1) == 0
    }
}
```

##### [292. Nim 游戏](https://leetcode.cn/problems/nim-game/description/)

> 简单题，思维题，简单来说就是谁拿的时候还剩4个，谁就肯定赢不了

```Swift
class Solution {
    func canWinNim(_ n: Int) -> Bool {
        return n % 4 != 0
    }
}
```

##### [344. 反转字符串](https://leetcode.cn/problems/reverse-string/description/)

> 简单题，使用元组或用swapAt函数都行

```Swift
class Solution {
    func reverseString(_ s: inout [Character]) {
        var l = 0, r = s.count - 1
        while l < r {
      	    // 使用元组
            (s[l], s[r]) = (s[r], s[l])
            l += 1
            r -= 1
        }
    }
}
```

```Swift
class Solution {
    func reverseString(_ s: inout [Character]) {
        var l = 0, r = s.count - 1
        while l < r {
            s.swapAt(l, r)
            l += 1
            r -= 1
        }
    }
}
```

##### [557. 反转字符串中的单词 III](https://leetcode.cn/problems/reverse-words-in-a-string-iii/description/)

> 简单题，直接用 split 和 joined 函数处理

```Swift
class Solution {
    func reverseWords(_ s: String) -> String {
        return s.split(separator: " ").map { String($0.reversed()) }.joined(separator: " ")
    }
}
```

##### [2. 两数相加](https://leetcode.cn/problems/add-two-numbers/description/)

> 中等题，注意进位即可

```Swift
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     public var val: Int
 *     public var next: ListNode?
 *     public init() { self.val = 0; self.next = nil; }
 *     public init(_ val: Int) { self.val = val; self.next = nil; }
 *     public init(_ val: Int, _ next: ListNode?) { self.val = val; self.next = next; }
 * }
 */
class Solution {
    func addTwoNumbers(_ l1: ListNode?, _ l2: ListNode?) -> ListNode? {
        let dummy = ListNode(-1)
        var cur: ListNode? = dummy
        var node1 = l1
        var node2 = l2
        var carry = false
        while node1 != nil || node2 != nil || carry {
            var sum = 0
            if let n1 = node1 {
                sum += n1.val
            }
            if let n2 = node2 {
                sum += n2.val
            }
            sum += carry ? 1 : 0
            carry = sum / 10 > 0
            let newNode = ListNode(sum % 10)
            cur?.next = newNode
            cur = cur?.next
            node1 = node1?.next
            node2 = node2?.next
        }
        return dummy.next
    }
}
```

##### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/description/)

> 中等题，用中心扩散法做，奇数用i自身向两边扩散，偶数用 i和i+1 两个数向两边扩撒

```Swift
class Solution {
    func longestPalindrome(_ s: String) -> String {
        let chars = Array(s)
        var start = 0, end = 0
        for i in 0..<chars.count {
            let len1 = expandAroundCenter(chars, i, i)
            let len2 = expandAroundCenter(chars, i, i + 1)
            let len = max(len1, len2)
            if (len > end - start) {
                start = i - (len - 1) / 2;
                end = i + len / 2;
            }
        }
        return String(chars[start..<end + 1])
    }

    func expandAroundCenter(_ chars: [Character], _ left: Int, _ right: Int) -> Int {
        var left = left, right = right
        while left >= 0 && right < chars.count {
            if chars[left] != chars[right] {
                break
            }
            left -= 1
            right += 1
        }
        return right - left - 1
    }
}
```

##### [7. 整数反转](https://leetcode.cn/problems/reverse-integer/description/)

> 中等题，不点评了，看代码

```Swift
class Solution {
    func reverse(_ x: Int) -> Int {
        var x = x
        var res = 0
        while x != 0 {
            let mod = x % 10
            x /= 10
            res = res * 10 + mod
        }
        return res < Int32.min || res > Int32.max ? 0 : res
    }
}
```

##### [8. 字符串转换整数 (atoi)](https://leetcode.cn/problems/string-to-integer-atoi/description/)

> 中等题，不点评，看代码

```Swift
class Solution {
    enum TOIState {
        case start, num, end
    }

    func myAtoi(_ s: String) -> Int {
        var characters: [Character] = []
        var state = TOIState.start
        var isNegative = false
        for c in s {
            if c.isWhitespace && state == .start {
                continue
            } else if c == "-" && state == .start {
                state = .num
                isNegative = true
            } else if c == "+" && state == .start {
                state = .num
                isNegative = false
            } else if c.isNumber {
                if state == .num {
                    characters.append(c)
                } else if state == .start {
                    state = .num
                    isNegative = false
                    characters.append(c)
                } else {
                    state = .end
                    break
                }
            } else {
                state = .end
                break
            }
        }
        guard characters.count > 0 else { return 0 }
        var num = Int(String(characters)) ?? Int.max
        if isNegative {
            num = -num
            if num < Int32.min {
                return Int(Int32.min)
            }
        }
        if num > Int32.max {
            return Int(Int32.max)
        }
        return num
    }
}
```

##### [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/description/)

> 中等题，双指针，简单理解就是那边小，哪边往中间挪一步

```Swift
class Solution {
    func maxArea(_ height: [Int]) -> Int {
        var result = 0, leftBar = 0, rightBar = height.count - 1
        while leftBar < rightBar {
            let area = (rightBar - leftBar) * min(height[leftBar], height[rightBar])
            result = max(result, area)
            if height[leftBar] < height[rightBar] {
                leftBar += 1
            } else {
                rightBar -= 1
            }
        }
        return result
    }
}
```

##### [15. 三数之和](https://leetcode.cn/problems/3sum/description/)

> 中等题，先排序，针对每个数，用双指针操作后面两个数遍历，注意同值的情况处理。

```Swift
class Solution {
        func threeSum(_ nums: [Int]) -> [[Int]] {
        guard nums.count > 2 else { return [] }
        var res: [[Int]] = []
        let sorted = nums.sorted()
        var i = 0
        while i < sorted.count - 2 {
            if i > 0, sorted[i] == sorted[i - 1] {
                i += 1
                continue
            }
            var l = i + 1
            var r = sorted.count - 1
            while l < r {
                let target = sorted[i] + sorted[l] + sorted[r]
                if (target == 0) {
                    if l - 1 > i, sorted[l] == sorted[l - 1] {
                        l += 1
                        continue
                    }
                    res.append([sorted[i], sorted[l], sorted[r]])
                    l += 1
                } else if (target < 0) {
                    l += 1
                } else {
                    r -= 1
                }
            }
            i += 1
        }
        return res
    }
}
```

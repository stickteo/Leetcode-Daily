# 2024-09-07
[1367. Linked List in Binary Tree](https://leetcode.com/problems/linked-list-in-binary-tree/)

Tried to compare it ad-hoc... The logic becomes complicated. (We only want to visit every node only once.) The second version would be the "naive" method but uses a lot of memory.

The way LeetCode measures the speed is all over the place... Got one run out of 6 beating 100% of submissions.

Presumably the fastest way is to have a buffer. To reduce memory usage, the solution should be iterative instead of recursive.
## Works

```C
bool dfs(uint8_t *pat, int pn, struct TreeNode* node, uint8_t *buf, int bn) {
    if(bn>=pn) {
        int i = 0;
        int j = bn-pn;
        while (i<pn) {
            if (pat[i] != buf[j]) {
                break;
            }
            i++;
            j++;
        }
        if (i==pn) {
            return true;
        }
    }

    if (node == NULL) {
        return false;
    }
    buf[bn] = node->val;
    if (dfs(pat,pn,node->left,buf,bn+1)) {
        return true;
    }
    if (dfs(pat,pn,node->right,buf,bn+1)) {
        return true;
    }

    return false;
}

bool isSubPath(struct ListNode* head, struct TreeNode* root) {
    uint8_t buf[2500];
    uint8_t pat[100];

    int pn = 0;
    while (head != NULL) {
        pat[pn] = head->val;
        pn++;
        head = head->next;
    }

    return dfs(pat,pn,root,buf,0);
}
```

## Fail

```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
bool dfs(struct ListNode *head, struct TreeNode* root, struct ListNode *curr) {
    if (curr == NULL) {
        return true;
    }
    if (root == NULL) {
        return false;
    }
    printf("%d ",curr->val);

    if (root->val != curr->val) {
        /*
        if (head != curr) {
            if (dfs(head,root,head)) {
                return true;
            }
            return false;
        }
        */
        if (head == curr) {
            if (dfs(head,root->left,head)) {
                return true;
            }
            if (dfs(head,root->right,head)) {
                return true;
            }
            return false;
        } else {
            if (dfs(head,root,head)) {
                return true;
            }
            return false;
        }
    } else {
        if (dfs(head,root->left,curr->next)) {
            return true;
        }
        if (dfs(head,root->right,curr->next)) {
            return true;
        }
    }
    return false;
}

bool isSubPath(struct ListNode* head, struct TreeNode* root) {
    return dfs(head,root,head);
}

```

# 2024-09-06
[3217. Delete Nodes From Linked List Present in Array](https://leetcode.com/problems/delete-nodes-from-linked-list-present-in-array/)

```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* modifiedList(int* nums, int numsSize, struct ListNode* head) {
    bool *b = calloc(100001,1);

    for (int i=0; i<numsSize; i++) {
        b[nums[i]] = true;
    }

    struct ListNode l;
    l.val = 0;
    l.next = head;

    struct ListNode *curr, *last;
    last = &l;
    curr = &l;

    while (curr) {
        if (!b[curr->val]) {
            last = curr;
        } else {
            last->next = curr->next;
        }
        curr = curr->next;
    }

    return l.next;
}
```
# 2024-09-05
[2028. Find Missing Observations](https://leetcode.com/problems/find-missing-observations/)

```Rust
impl Solution {
    pub fn missing_rolls(rolls: Vec<i32>, mean: i32, n: i32) -> Vec<i32> {
        let count = rolls.len() + n as usize;
        let total = count as i32 * mean;

        let left = total - rolls.iter().sum::<i32>();
        if left < n {
            return Vec::new();
        }
        if left > n*6 {
            return Vec::new();
        }
        let b = left/n;
        let r = (left%n) as usize;

        let mut out = vec![b;n as usize];
        for i in 0..r {
            out[i] += 1;
        }
        out
    }
}
```

# 2024-09-04
[874. Walking Robot Simulation](https://leetcode.com/problems/walking-robot-simulation/)

Doesn't look pretty but it works.

```Rust
use std::collections::HashMap;

impl Solution {
    pub fn robot_sim(commands: Vec<i32>, obstacles: Vec<Vec<i32>>) -> i32 {
        let mut xobs = HashMap::new();
        let mut yobs = HashMap::new();

        for e in obstacles.iter() {
            let x = e[0];
            let y = e[1];
            match xobs.get_mut(&x) {
                None => {
                    let mut a = Vec::new();
                    a.push(y);
                    xobs.insert(x,a);
                }
                Some(a) => {
                    a.push(y);
                    a.sort();
                }
            }
            match yobs.get_mut(&y) {
                None => {
                    let mut a = Vec::new();
                    a.push(x);
                    yobs.insert(y,a);
                }
                Some(a) => {
                    a.push(x);
                    a.sort();
                }
            }
        }

        let mut dir = 0;
        let mut x = 0;
        let mut y = 0;
        let mut max = 0;

        for c in commands.iter() {
            match c {
                -2 => {
                    dir = (dir+3)%4;
                }
                -1 => {
                    dir = (dir+1)%4;
                }
                1..=9 => {
                    match dir {
                        0 => {
                            //print!("n ");
                            match xobs.get(&x) {
                                None => {
                                    y += c;
                                }
                                Some(a) => {
                                    let mut i = 0;
                                    let mut yn = y+c;
                                    while i<a.len() && y>=a[i] {
                                        i += 1;
                                    }
                                    if i<a.len() && yn>a[i]-1 {
                                        yn = a[i]-1;
                                    }
                                    y = yn;
                                }
                            }
                        }
                        1 => {
                            //print!("e ");
                            match yobs.get(&y) {
                                None => {
                                    x += c;
                                }
                                Some(a) => {
                                    let mut i = 0;
                                    let mut xn = x+c;
                                    while i<a.len() && x>=a[i] {
                                        i += 1;
                                    }
                                    if i<a.len() && xn>a[i]-1 {
                                        xn = a[i]-1;
                                    }
                                    x = xn;
                                }
                            }
                        }
                        2 => {
                            //print!("s ");
                            match xobs.get(&x) {
                                None => {
                                    y -= c;
                                }
                                Some(a) => {
                                    let mut i = a.len()-1;
                                    let mut yn = y-c;
                                    while i<a.len() && y<=a[i] {
                                        i -= 1;
                                    }
                                    if i<a.len() && yn<a[i]+1 {
                                        yn = a[i]+1;
                                    }
                                    y = yn;
                                }
                            }
                        }
                        3 => {
                            //print!("w ");
                            match yobs.get(&y) {
                                None => {
                                    x -= c;
                                }
                                Some(a) => {
                                    let mut i = a.len()-1;
                                    let mut xn = x-c;
                                    while i<a.len() && x<=a[i] {
                                        i -= 1;
                                    }
                                    if i<a.len() && xn<a[i]+1 {
                                        xn = a[i]+1;
                                    }
                                    x = xn;
                                }
                            }
                        }
                        _ => ()
                    }
                    //print!("{},{} ",x,y);
                    let dis = x*x+y*y;
                    if dis>max {
                        max = dis;
                    }
                }
                _ => ()
            }
        }

        /*
        println!("{},{}",x,y);
        println!("x {:?}",xobs);
        println!("y {:?}",yobs);
        */

        max
    }
}
```
# 2024-09-03
[1945. Sum of Digits of String After Convert](https://leetcode.com/problems/sum-of-digits-of-string-after-convert/)

```Rust
impl Solution {
    pub fn get_lucky(s: String, k: i32) -> i32 {
        let a:Vec<String> = s.chars()
            .map(|x| format!("{}",x as u8-b'a'+1))
            .collect();
        let mut b = a.concat();
        let mut c = 0;

        for i in 0..k {
            c = digit_sum(b);
            b = format!("{}",c);
        }

        c
    }
}

fn digit_sum(s: String) -> i32 {
    s.bytes().map(|x| (x-b'0') as i32).sum()
}
```

# 2024-09-02
[1894. Find the Student that Will Replace the Chalk](https://leetcode.com/problems/find-the-student-that-will-replace-the-chalk/)

```Rust
impl Solution {
    pub fn chalk_replacer(chalk: Vec<i32>, k: i32) -> i32 {
        let total:i64 = chalk.iter().map(|x| *x as i64).sum();
        let mut k = if (k as i64)<total {
            k
        } else {
            k%(total as i32)
        };
        let mut i = 0;
        while chalk[i]<=k {
            k -= chalk[i];
            i += 1;
        }

        i as i32
    }
}
```

# 2024-09-01
[2022. Convert 1D Array Into 2D Array](https://leetcode.com/problems/convert-1d-array-into-2d-array/)

```Rust
impl Solution {
    pub fn construct2_d_array(original: Vec<i32>, m: i32, n: i32) -> Vec<Vec<i32>> {
        if (n*m) as usize != original.len() {
            return Vec::new();
        }
        original[..].chunks_exact(n as usize).map(|x| x.try_into().unwrap()).collect()
    }
}
```

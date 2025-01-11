# 2024-12-30
[2466. Count Ways To Build Good Strings](https://leetcode.com/problems/count-ways-to-build-good-strings/)

## TLE (raw convolution)
```Rust
impl Solution {
    pub fn count_good_strings(low: i32, high: i32, zero: i32, one: i32) -> i32 {
        let zero = zero as usize;
        let one = one as usize;
        let low = low as usize;
        let high = high as usize;
        let mut base = zero.min(one);
        let mut vals = vec![0; (zero as i32-one as i32).abs() as usize+1];
        vals[zero-base] += 1;
        vals[one-base] += 1;

        let mut count = 0 as i64;
        let mut basec = base;
        let mut valsc = vals.clone();

        while basec <= high {
            println!("{} {:?}",basec,valsc);
            if low < basec+valsc.len() {
                for i in low.max(basec)-basec..(high+1-basec).min(valsc.len()) {
                    count += valsc[i] as i64;
                    count %= 1000000007;
                }
            }
            let mut valsn = vec![0; valsc.len()+vals.len()-1];
            for i in 0..vals.len() {
                for j in 0..valsc.len() {
                    valsn[i+j] = ((vals[i] as i64 * valsc[j] as i64 + valsn[i+j] as i64)%1000000007) as i32;
                }
            }
            basec += base;
            valsc = valsn;
        }

        //println!("{:?}",vals);
        count as i32
    }
}

```
# 2024-12-29
[1639. Number of Ways to Form a Target String Given a Dictionary](https://leetcode.com/problems/number-of-ways-to-form-a-target-string-given-a-dictionary/)

Yet another dynamic programming problem... The general theme is that we require using a lot of memory to speed up computation. Then the problem becomes a sort of puzzle to figure out how to calculate things.

```Rust
impl Solution {
    pub fn num_ways(words: Vec<String>, target: String) -> i32 {
        if target.len()>words[0].len() {
            return 0;
        }

        let mut freq = vec![vec![0; 26]; words[0].len()];
        for w in words.iter() {
            for (i,a) in w.bytes().enumerate() {
                let a = (a-b'a') as usize;
                freq[i][a] += 1;
            }
        }
        //println!("{:?}",freq);

        let target: Vec<_> = target.bytes().map(|x| (&x-b'a') as usize).collect();

        let mut dp = vec![vec![-1; words[0].len()]; target.len()];
        calc(&freq,&mut dp,&target,0,0);
        //println!("{:?}",dp);
        dp[0][0]
    }
}

fn calc(freq: &Vec<Vec<i32>>, dp: &mut Vec<Vec<i32>>, target: &Vec<usize>, t: usize, w: usize) -> i32 {
    if t >= dp.len() {
        return 1;
    }
    if w >= dp[0].len() {
        return 0;
    }
    if dp[t][w] != -1 {
        return dp[t][w];
    }

    let mut count = 0;
    let a = freq[w][target[t]] as i64;
    if a>0 {
        count += a*(calc(freq,dp,target,t+1,w+1) as i64);
    }
    count += calc(freq,dp,target,t,w+1) as i64;
    dp[t][w] = (count%1000000007) as i32;
    dp[t][w]
}
}
```

# 2024-12-28
[689. Maximum Sum of 3 Non-Overlapping Subarrays](https://leetcode.com/problems/maximum-sum-of-3-non-overlapping-subarrays/)

Works but not very efficient.

```Rust
impl Solution {
    pub fn max_sum_of_three_subarrays(nums: Vec<i32>, k: i32) -> Vec<i32> {
        let k = k as usize;
        let mut sum = 0;
        for i in 0..k {
            sum += nums[i];
        }
        let mut sums = Vec::new();
        sums.push(sum);
        for i in k..nums.len() {
            sum -= nums[i-k];
            sum += nums[i];
            sums.push(sum);
        }
        //println!("{:?}",sums);
        let mut stackv = Vec::new();
        let mut stacki = Vec::new();
        //stack.push(vec![(sums.len()-1,*sums.last().unwrap())]);
        for i in (0..sums.len()).rev() {
            match stackv.last() {
                None => {
                    stacki.push(vec![i]);
                    stackv.push(sums[i]);
                }
                Some(x) => {
                    if sums[i]>=*x {
                        stacki.push(vec![i]);
                        stackv.push(sums[i]);
                    }
                }
            }
        }
        let mut stack: Vec<_> = stacki.iter().zip(stackv.iter()).collect();
        //println!("{:?}",stack);

        let mut stacki = Vec::new();
        let mut stackv = Vec::new();
        let mut i = 0;
        while i<sums.len() {
            match stack.last() {
                None => {
                    break;
                }
                Some(x) => {
                    if i+k<=x.0[0] {
                        let mut a = vec![i];
                        a.extend(x.0);
                        stacki.push(a);
                        stackv.push(sums[i]+x.1);
                        i += 1;
                    } else {
                        stack.pop();
                    }
                }
            }
        }
        //let mut stack: Vec<_> = stacki.iter().zip(stackv.iter()).collect();
        //println!("{:?}",stack);
        let mut stack = Vec::new();
        for i in (0..stackv.len()).rev() {
            match stack.last() {
                None => {
                    stack.push((stacki[i].clone(),stackv[i]));
                }
                Some(x) => {
                    if stackv[i]>=x.1 {
                        stack.push((stacki[i].clone(),stackv[i]));
                    }
                }
            }
        }
        //println!("{:?}",stack);

        let mut stacki = Vec::new();
        let mut stackv = Vec::new();
        let mut i = 0;
        while i<sums.len() {
            match stack.last() {
                None => {
                    break;
                }
                Some(x) => {
                    if i+k<=x.0[0] {
                        let mut a = vec![i];
                        a.extend(&x.0);
                        stacki.push(a);
                        stackv.push(sums[i]+x.1);
                        i += 1;
                    } else {
                        stack.pop();
                    }
                }
            }
        }
        let mut stack = Vec::new();
        for i in (0..stackv.len()).rev() {
            match stack.last() {
                None => {
                    stack.push((stacki[i].clone(),stackv[i]));
                }
                Some(x) => {
                    if stackv[i]>=x.1 {
                        stack.push((stacki[i].clone(),stackv[i]));
                    }
                }
            }
        }
        //println!("{:?}",stack);

        stack.last().unwrap().0.iter().map(|x| *x as i32).collect()
    }
}
```

# 2024-12-27
[1014. Best Sightseeing Pair](https://leetcode.com/problems/best-sightseeing-pair/)

~~Not the best approach but works okay.~~ The speed can be optimized so now it runs in O(n) time. We can even further remove an array so it should be O(1) space.

## Space Optimized
```Rust
impl Solution {
    pub fn max_score_sightseeing_pair(values: Vec<i32>) -> i32 {
        let mut max = 0;
        let mut b = values[0]-1;
        for i in 1..values.len() {
            let v = b+values[i];
            if max<v {
                max = v;
            }
            if b<values[i] {
                b = values[i]-1;
            } else {
                b -= 1;
            }
        }
        max
    }
}
```

## Time Optimized
```Rust
impl Solution {
    pub fn max_score_sightseeing_pair(values: Vec<i32>) -> i32 {
        let mut best = vec![0; values.len()];
        let mut b = values[0]-1;
        for i in 1..values.len() {
            best[i] = b;
            if b<values[i] {
                b = values[i]-1;
            } else {
                b -= 1;
            }
        }
        let mut max = 0;
        for i in 1..values.len() {
            let b = values[i]+best[i];
            if max<b {
                max = b;
            }
        }
        max
    }
}
```

## Unoptimized
```Rust
impl Solution {
    pub fn max_score_sightseeing_pair(values: Vec<i32>) -> i32 {
        let mut best = vec![0; values.len()];
        for (i,v) in values.iter().enumerate() {
            let mut v = *v-1;
            for j in i+1..values.len() {
                if v<=best[j] {
                    break;
                }
                best[j] = v;
                v -= 1;
            }
        }
        //println!("{:?}",best);
        let mut max = 0;
        for i in 1..values.len() {
            let b = values[i]+best[i];
            if max<b {
                max = b;
            }
        }
        max
    }
}
```

# 2024-12-26
[494. Target Sum](https://leetcode.com/problems/target-sum/)

Convolution method.

Two ways to manage the vectors... Either allocate a new "next" to update "curr" or to swap each while zeroing them.

~~Both seem equally fast (both measures 0ms) but swapping uses slightly less memory. (Well, 0.07MB less memory.)~~

~~Ideally both should be the same and compile to the same assembly... But there is a difference... ~~

Memory use differences seem to be within error with having 3 runs of each.

```Rust
impl Solution {
    pub fn find_target_sum_ways(nums: Vec<i32>, target: i32) -> i32 {
        let mut curr = vec![0; 2001];
        let mut next = vec![0; 2001];
        curr[1000] = 1;

        for e in nums.iter() {
            let e = *e as usize;
            for i in 0..2001-e {
                next[i] += curr[i+e];
            }
            for i in e..2001 {
                next[i] += curr[i-e];
            }
            for i in 0..2001 {
                curr[i] = 0;
            }
            let mut temp = curr;
            curr = next;
            next = temp;
        }

        curr[target as usize+1000]
    }
}
```

```Rust
impl Solution {
    pub fn find_target_sum_ways(nums: Vec<i32>, target: i32) -> i32 {
        let mut curr = vec![0; 2001];
        curr[1000] = 1;

        for e in nums.iter() {
            let e = *e as usize;
            let mut next = vec![0; 2001];
            for i in 0..2001-e {
                next[i] += curr[i+e];
            }
            for i in e..2001 {
                next[i] += curr[i-e];
            }
            curr = next;
        }

        curr[target as usize+1000]
    }
}
```

# 2024-12-25
[515. Find Largest Value in Each Tree Row](https://leetcode.com/problems/find-largest-value-in-each-tree-row/)

The DFS approach is considerably faster. Indeed, the DFS approach is O(n) where n is the amount of nodes. For the BFS approach, it's O(n log(n)).

## DFS
```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

int tree_size(struct TreeNode* node) {
    if (!node) {
        return 0;
    }
    int size = 1;
    size += tree_size(node->left);
    size += tree_size(node->right);
    return size;
}

int tree_depth(struct TreeNode* node) {
    if (!node) {
        return 0;
    }
    int l = 1 + tree_depth(node->left);
    int r = 1 + tree_depth(node->right);
    if (l>r) {
        return l;
    } else {
        return r;
    }
}

void tree_max(struct TreeNode* node, int* vals, int d) {
    if (!node) {
        return;
    }
    if (vals[d]<node->val) {
        vals[d] = node->val;
    }
    tree_max(node->left,vals,d+1);
    tree_max(node->right,vals,d+1);
}

int* largestValues(struct TreeNode* root, int* returnSize) {
    if (!root) {
        *returnSize = 0;
        return NULL;
    }
    int n = tree_size(root);
    int d = tree_depth(root);
    int *out = malloc(d*sizeof(int));
    for (int i=0; i<d; i++) {
        out[i] = INT_MIN;
    }
    tree_max(root,out,0);
    *returnSize = d;
    return out;
}
```

## BFS
```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

int tree_size(struct TreeNode* node) {
    if (!node) {
        return 0;
    }
    int size = 1;
    size += tree_size(node->left);
    size += tree_size(node->right);
    return size;
}

int tree_depth(struct TreeNode* node) {
    if (!node) {
        return 0;
    }
    int l = 1 + tree_depth(node->left);
    int r = 1 + tree_depth(node->right);
    if (l>r) {
        return l;
    } else {
        return r;
    }
}

int* largestValues(struct TreeNode* root, int* returnSize) {
    if (!root) {
        *returnSize = 0;
        return NULL;
    }
    int n = tree_size(root);
    int d = tree_depth(root);
    struct TreeNode** curr = malloc(n*sizeof(struct TreeNode*));
    struct TreeNode** next = malloc(n*sizeof(struct TreeNode*));
    int *out = malloc(d*sizeof(int));
    int m = 1;
    int j = 0;
    curr[0] = root;

    while (m>0) {
        int k=0;
        int max = INT_MIN;
        for (int i=0; i<m; i++) {
            if (curr[i]->left) {
                next[k] = curr[i]->left;
                k++;
            }
            if (curr[i]->right) {
                next[k] = curr[i]->right;
                k++;
            }
            if (max<curr[i]->val) {
                max = curr[i]->val;
            }
        }
        out[j] = max;
        m = k;
        struct TreeNode** temp = curr;
        curr = next;
        next = temp;
        j++;
    }

    *returnSize = d;
    return out;
}
```

# 2024-12-24
[3203. Find Minimum Diameter After Merging Two Trees](https://leetcode.com/problems/find-minimum-diameter-after-merging-two-trees/)

Fairly simple logic. The "radius" of the tree can be found by trimming the branches until you get to the "core". (At the same time also finding the core.) The core can either be 1 or 2 nodes. If one node, then the diameter is simply twice the radius. Else if two, it's twice the radius minus one.

From there, if one tree has a really large radius, adding a small tree to it won't affect its diameter. So we compare the sum of the radius of both trees to both the diameters of both trees.

Maybe there's a faster way where you can get the diameter directly without trimming...

```Rust
use std::collections::HashSet;

impl Solution {
    pub fn minimum_diameter_after_merge(edges1: Vec<Vec<i32>>, edges2: Vec<Vec<i32>>) -> i32 {
        let (r1,d1) = diameter(&edges1);
        let (r2,d2) = diameter(&edges2);
        let a = vec![r1+r2+1,d1,d2];
        *a.iter().max().unwrap()
    }
}

fn num_nodes(edges: &Vec<Vec<i32>>) -> usize {
    let mut max = 0;
    for e in edges.iter() {
        if e[0]>max {
            max = e[0];
        }
        if e[1]>max {
            max = e[1];
        }
    }
    (max+1) as usize
}

fn diameter(edges: &Vec<Vec<i32>>) -> (i32,i32) {
    //let mut count = vec![0; edges.len()];
    if edges.len()==0 {
        return (0,0);
    }
    let n = num_nodes(edges);
    let mut neighbors = vec![HashSet::new(); n];
    for e in edges.iter() {
        let a = e[0] as usize;
        let b = e[1] as usize;
        //count[a]+=1;
        //count[b]+=1;
        neighbors[a].insert(b);
        neighbors[b].insert(a);
    }
    let mut curr = HashSet::new();
    for (i,e) in neighbors.iter().enumerate() {
        if e.len() == 1 {
            curr.insert(i);
        }
    }
    let mut r = 0;
    let mut count = n;
    while !curr.is_empty() {
        //println!("{:?}",curr);
        r += 1;
        let mut next = HashSet::new();
        let mut remove = Vec::new();
        for &i in curr.iter() {
            if neighbors[i].len() > 1 {
                continue;
            }
            count -= 1;
            if neighbors[i].len() == 0 {
                continue;
            }
            let j = *neighbors[i].iter().next().unwrap();
            //neighbors[j].remove(&i);
            remove.push((i,j));
            next.insert(j);
        }
        for (i,j) in remove.into_iter() {
            neighbors[j].remove(&i);
        }
        curr = next;
    }
    r -= 1;
    let mut d = r*2;
    if count>1 {
        d -= 1;
    }
    //println!("{} {}",r,d);
    (r,d)
}
```

# 2024-12-23
[2471. Minimum Number of Operations to Sort a Binary Tree by Level](https://leetcode.com/problems/minimum-number-of-operations-to-sort-a-binary-tree-by-level/)

Painful to implement in C. Two times slower than the fastest. Luckily not having duplicate values make it easier.

The problem really gets split into two parts... Managing the tree, which was seemingly simple. The other is finding the optimal amount of swaps... We can do it without fancy data structures by continually swapping our current element into its ideal position... Then the swapped element will be our next current element. We loop until we have the ideal element in our position then we move onto the next position.

Perhaps the alternative is not doing any swaps but we simply search until we create a loop with the ideal element for the current position. We would need to make which spots are reached though.

```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
int depth(struct TreeNode* node) {
    if (node == NULL) {
        return 0;
    }
    int l = 1+depth(node->left);
    int r = 1+depth(node->right);
    if (l<r) {
        return r;
    } else {
        return l;
    }
}

int tree_size(struct TreeNode* node) {
    if (node == NULL) {
        return 0;
    }
    int s = 1;
    s += tree_size(node->left);
    s += tree_size(node->right);
    return s;
}

int compare(const void *a, const void *b) {
    return *(int*)a-*(int*)b;
}

int num_ops(int len, int *bufa, int *bufb) {
    for(int i=0; i<len; i++) {
        bufb[i] = bufa[i];
    }
    qsort(bufb,len,sizeof(int),compare);
    int swaps = 0;
    for(int i=0; i<len; i++) {
        if (bufa[i]==bufb[i]) {
            continue;
        }
        //int pos = (bsearch(&bufa[i],bufb,len,sizeof(int),compare)-(void*)bufb)/sizeof(int);
        int pos = (int*)bsearch(&bufa[i],bufb,len,sizeof(int),compare)-bufb;
        while (pos!=i) {
            int t = bufa[i];
            bufa[i] = bufa[pos];
            bufa[pos] = t;
            pos = (int*)bsearch(&bufa[i],bufb,len,sizeof(int),compare)-bufb;
            swaps++;
        }
    }
    return swaps;
}

int minimumOperations(struct TreeNode* root) {
    int d = depth(root);
    int s = tree_size(root);
    //printf("%d\n",d);
    struct TreeNode **curr = malloc(s*sizeof(struct TreeNode*));
    struct TreeNode **next = malloc(s*sizeof(struct TreeNode*));
    int *bufa = malloc(s*sizeof(int));
    int *bufb = malloc(s*sizeof(int));
    int n = 1;
    int lvl = 0;
    int swaps = 0;
    curr[0] = root;
    while (lvl<d) {
        for(int i=0; i<n; i++) {
            bufa[i] = curr[i]->val;
            //printf("%d ",curr[i]->val);
        }
        //printf("\n");
        swaps += num_ops(n, bufa, bufb);
        /*
        for(int i=0; i<n; i++) {
            printf("%d ",bufb[i]);
        }
        printf("\n");
        for(int i=0; i<n; i++) {
            printf("%d ",bufa[i]);
        }
        printf("\n");
        */
        int m = 0;
        for(int i=0; i<n; i++) {
            if (curr[i]->left) {
                next[m] = curr[i]->left;
                m++;
            }
            if (curr[i]->right) {
                next[m] = curr[i]->right;
                m++;
            }
        }
        n = m;
        lvl++;
        struct TreeNode **temp = curr;
        curr = next;
        next = temp;
    }
    return swaps;
}
```

# 2024-12-22
[2940. Find Building Where Alice and Bob Can Meet](https://leetcode.com/problems/find-building-where-alice-and-bob-can-meet/)

Followed the hints for this one. There are several key parts that needed to be figured out... The wording of the question can be better... To my own credit, I figured out the first 3 hints. The last hint basically tells you how to do the problem. The implementation is also technically difficult to get correct.

```Rust
impl Solution {
    pub fn leftmost_building_queries(heights: Vec<i32>, queries: Vec<Vec<i32>>) -> Vec<i32> {
        let mut q = Vec::new();
        let mut out = vec![-1; queries.len()];
        for (i,e) in queries.iter().enumerate() {
            let a = e[0] as usize;
            let b = e[1] as usize;
            let n = a.max(b);
            let h = heights[a].max(heights[b]);
            if h == heights[n] && (a == b || heights[a] != heights[b]) {
                out[i] = n as i32;
            } else {
                q.push((n,h,i));
            }
        }
        q.sort_unstable();
        //println!("{:?}",q);

        let mut stack = Vec::new();
        let mut p = heights.len()-1;

        while let Some((n,h,i)) = q.pop() {
            while p>n {
                match stack.last() {
                    None => {
                        stack.push(p);
                        p -= 1;
                    }
                    Some(&x) => {
                        if heights[x]<=heights[p] {
                            stack.pop();
                        } else {
                            stack.push(p);
                            p -= 1;
                        }
                    }
                }
            }
            //println!("({},{},{})",n,h,i);
            //println!("{:?}", stack);
            //for k in stack.iter() {
            //    print!("{} ",heights[*k]);
            //}
            //println!();
            let s = stack.partition_point(|&x| heights[x]>h);
            //println!("{} {}",s,h);
            if s==0 {
                out[i] = -1;
                continue;
            }
            let j = stack[s-1];
            if j>=heights.len() {
                out[i] = -1;
            } else {
                out[i] = j as i32;
            }
        }

        out
    }
}
```

# 2024-12-21
[2872. Maximum Number of K-Divisible Components](https://leetcode.com/problems/maximum-number-of-k-divisible-components/)

Not too difficult... The implementation can get quite technical...

The main idea is "trimming" the ends of the tree down. It's like a BFS but you aim for nodes with a single edge... Hence the "ends" of the tree.

Not the fastest but not using any particularly fancy data structures... Just vectors are used.

```Rust
impl Solution {
    pub fn max_k_divisible_components(n: i32, edges: Vec<Vec<i32>>, values: Vec<i32>, k: i32) -> i32 {
        if k==1 {
            return n;
        }
        let n = n as usize;
        if n==1 && values[0]%k==0 {
            return 1;
        }
        let mut count = vec![0; n];
        let mut neighbors = vec![Vec::new(); n];
        let mut values = values;
        for e in edges.iter() {
            let a = e[0] as usize;
            let b = e[1] as usize;
            count[a] += 1;
            count[b] += 1;
            neighbors[a].push(b);
            neighbors[b].push(a);
        }
        //println!("{:?}",neighbors);
        //println!("{:?}",count);

        let mut curr = Vec::new();
        for i in 0..n {
            if count[i] == 1 {
                curr.push(i);
            }
            values[i] %= k;
        }
        let mut out = 0;
        let mut reach = vec![false; n];
        while !curr.is_empty() {
            //println!("{:?}",curr);
            let mut next = Vec::new();
            for i in curr.into_iter() {
                if reach[i] {
                    continue;
                }
                if count[i] > 1 {
                    continue;
                }
                reach[i] = true;
                if count[i] == 0 {
                    //print!("{} ",i);
                    out += 1;
                    continue;
                }
                let j = neighbors[i][0];
                if let Some(pos) = neighbors[j].iter().position(|x| *x==i) {
                    neighbors[j].swap_remove(pos);
                }
                count[i] -= 1;
                count[j] -= 1;
                if values[i] == 0 {
                    //print!("{} ",i);
                    out += 1;
                } else {
                    values[j] += values[i];
                    values[j] %= k;
                }
                reach[i] = true;
                next.push(j);
            }
            curr = next;
        }
        out
    }
}
```

# 2024-12-20
[2415. Reverse Odd Levels of Binary Tree](https://leetcode.com/problems/reverse-odd-levels-of-binary-tree/)

Fast but not memory efficient.

```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
int levels(struct TreeNode* node) {
    int l = 0;
    while (node) {
        node = node->left;
        l++;
    }
    return l;
}

struct TreeNode* reverseOddLevels(struct TreeNode* root) {
    int l = levels(root);
    //printf("%d\n",l);
    struct TreeNode** curr = malloc((1<<l)*sizeof(struct TreeNode*));
    struct TreeNode** next = malloc((1<<l)*sizeof(struct TreeNode*));
    curr[0] = root;
    int cl = 0;
    while (cl<l) {
        if (cl&1) {
            int k = 0;
            while (k<(1<<(cl-1))) {
                int t = curr[k]->val;
                curr[k]->val = curr[(1<<cl)-k-1]->val;
                curr[(1<<cl)-k-1]->val = t;
                k++;
            }
        }
        int i = 0;
        int j = 0;
        while (i<(1<<cl)) {
            next[j] = curr[i]->left;
            j++;
            next[j] = curr[i]->right;
            j++;
            i++;
        }
        struct TreeNode** temp = curr;
        curr = next;
        next = temp;
        cl++;
    }
    return root;
}
```
# 2024-12-19
[769. Max Chunks To Make Sorted](https://leetcode.com/problems/max-chunks-to-make-sorted/)

The two key insights is that 0 needs to be a part of the first chunk and each chunk needs to have consecutive values.

The problem's constraints make it easier to solve.

```Rust
impl Solution {
    pub fn max_chunks_to_sorted(arr: Vec<i32>) -> i32 {
        let mut chunks = 1;
        let mut min = arr[0];
        let mut max = arr[0];
        let mut target = 0;
        let mut count = 1;
        let mut i = 1;
        while i<arr.len() {
            if target>=min && target<=max && count==max-min+1 {
                target = max+1;
                min = arr[i];
                max = arr[i];
                count = 1;
                chunks += 1;
                i += 1;
                continue;
            }
            if arr[i]<min {
                min = arr[i];
            }
            if arr[i]>max {
                max = arr[i];
            }
            count += 1;
            i += 1;
        }
        chunks
    }
}
```

# 2024-12-18
[1475. Final Prices With a Special Discount in a Shop](https://leetcode.com/problems/final-prices-with-a-special-discount-in-a-shop/)

The hint simply says to use brute-force... In this case I try to optimize it... There are only 500 elements max... So a brute-force approach would mean around 500^2/2 comparisons max...

The stack-based approach is still decently fast... Tested with 0ms... The worst case (elements are in ascending order) would create a stack with N elements. Otherwise it should run in linear time.
## Stack-based
```Rust
impl Solution {
    pub fn final_prices(prices: Vec<i32>) -> Vec<i32> {
        let mut prices = prices;
        let mut i = 0;
        let mut stack = Vec::new();
        while i<prices.len() {
            match stack.last() {
                None => {
                    stack.push(i);
                    i += 1;
                }
                Some(&k) => {
                    if prices[i]>prices[k] {
                        stack.push(i);
                        i += 1;
                    } else {
                        stack.pop();
                        prices[k] -= prices[i];
                    }
                }
            }
        }
        prices
    }
}
```

## Attempt
```Rust
impl Solution {
    pub fn final_prices(prices: Vec<i32>) -> Vec<i32> {
        let mut prices = prices;
        let mut i = 0;
        while i<prices.len() {
            let mut j = i+1;
            while j<prices.len() && prices[j]>prices[i] {
                j += 1;
            }
            if j>=prices.len() {
                break;
            }
            while i<j {
                prices[i] -= prices[j];
                i += 1;
            }
        }
        prices
    }
}
```

# 2024-12-17
[2182. Construct String With Repeat Limit](https://leetcode.com/problems/construct-string-with-repeat-limit/)

```Rust
impl Solution {
    pub fn repeat_limited_string(s: String, repeat_limit: i32) -> String {
        let mut count = vec![0; 26];
        for e in s.as_bytes().iter() {
            count[(*e-b'a') as usize] += 1;
        }
        let mut a = Vec::new();
        for (i,&e) in count.iter().enumerate() {
            if e!=0 {
                a.push((i,e));
            }
        }
        //println!("{:?}",a);
        let mut b = String::new();
        let mut curr = a.pop().unwrap();
        while a.len()>0 {
            if curr.1 > repeat_limit {
                curr.1 -= repeat_limit;
                for i in 0..repeat_limit {
                    b.push((curr.0 as u8+b'a') as char);
                }
                if let Some(x) = a.last_mut() {
                    b.push((x.0 as u8+b'a') as char);
                    x.1 -= 1;
                    if x.1 == 0 {
                        a.pop();
                    }
                }
            } else {
                for i in 0..curr.1 {
                    b.push((curr.0 as u8+b'a') as char);
                }
                curr = a.pop().unwrap();
            }
        }
        if b.len()==0 || (b.as_bytes()[b.len()-1] != curr.0 as u8+b'a') {
            let c = repeat_limit.min(curr.1);
            for i in 0..c {
                b.push((curr.0 as u8+b'a') as char);
            }
        }
        b
    }
}
```
# 2024-12-16
[3264. Final Array State After K Multiplication Operations I](https://leetcode.com/problems/final-array-state-after-k-multiplication-operations-i/)

```Rust
use std::collections::BinaryHeap;
impl Solution {
    pub fn get_final_state(nums: Vec<i32>, k: i32, multiplier: i32) -> Vec<i32> {
        let mut nums = nums;
        let mut heap = BinaryHeap::new();
        for (i,n) in nums.iter().enumerate() {
            heap.push((0-n,0-i as i32));
        }
        let mut k = k;
        while k>0 {
            let e = heap.pop().unwrap();
            let n = (0-e.0)*multiplier;
            let i = 0-e.1 as usize;
            nums[i] = n;
            heap.push((0-n,0-i as i32));
            k -= 1;
        }
        nums
    }
}
```

# 2024-12-14
[2762. Continuous Subarrays](https://leetcode.com/problems/continuous-subarrays/)

Got it first try. Perhaps a bit complicated using a BTreeMap...

Checking the hints after solving it, I literally did what the hints told me... Sliding window and checking minimums and maximums... Should roughly run in O(n logn) time...

```Rust
use std::collections::BTreeMap;
impl Solution {
    pub fn continuous_subarrays(nums: Vec<i32>) -> i64 {
        let mut i = 0;
        let mut j = 0;
        let mut map = BTreeMap::new();
        let mut sum = 0;
        while j<nums.len() {
            map.entry(nums[j]).and_modify(|x| *x+=1).or_insert(1);
            let mut lo = *map.first_entry().unwrap().key();
            let mut hi = *map.last_entry().unwrap().key();
            while hi-lo>2 {
                map.entry(nums[i]).and_modify(|x| *x-=1).or_insert(0);
                if map[&nums[i]] == 0 {
                    map.remove(&nums[i]);
                }
                lo = *map.first_entry().unwrap().key();
                hi = *map.last_entry().unwrap().key();
                i += 1;
            }
            sum += (j-i+1) as i64;
            j += 1;
        }
        sum
    }
}
```

# 2024-12-13
[2593. Find Score of an Array After Marking All Elements](https://leetcode.com/problems/find-score-of-an-array-after-marking-all-elements/)

An easy problem. Just sorting and keeping a boolean array. No need for a heap... We don't need to dynamically sort items. (Using a heap implies sorting.)

## Simple
```Rust
impl Solution {
    pub fn find_score(nums: Vec<i32>) -> i64 {
        let mut b = vec![false; nums.len()];
        let mut a: Vec<(i32,i32)> = nums.into_iter().enumerate()
            .map(|x| (x.1,x.0 as i32)).collect();
        a.sort_unstable();
        let mut c = 0;
        for e in a.into_iter() {
            let i = e.1 as usize;
            let s = e.0 as i64;
            if b[i] {
                continue;
            }
            c += s;
            b[i] = true;
            if i>0 {
                b[i-1] = true;
            }
            if i<b.len()-1 {
                b[i+1] = true;
            }
        }
        c
    }
}
```

## Using Heap
```Rust
use std::collections::BinaryHeap;
impl Solution {
    pub fn find_score(nums: Vec<i32>) -> i64 {
        let mut marked = vec![false; nums.len()];
        let mut heap = BinaryHeap::new();
        for (i,s) in nums.into_iter().enumerate() {
            heap.push((0-s,0-i as i32));
        }
        let mut sum = 0;
        while let Some(e) = heap.pop() {
            let i = (0-e.1) as usize;
            let s = 0-e.0;
            //print!("({},{}) ",s,i);
            if marked[i] {
                continue;
            }
            sum += s as i64;
            marked[i] = true;
            if i>0 {
                marked[i-1] = true;
            }
            if i<marked.len()-1 {
                marked[i+1] = true;
            }
        }
        sum
    }
}
```

# 2024-12-12
[2558. Take Gifts From the Richest Pile](https://leetcode.com/problems/take-gifts-from-the-richest-pile/)

Straight forward... Heap and cast as a float to implement square root. Fast but not the best memory usage...

```Rust
use std::collections::BinaryHeap;

impl Solution {
    pub fn pick_gifts(gifts: Vec<i32>, k: i32) -> i64 {
        let mut heap = BinaryHeap::new();
        for g in gifts.iter() {
            heap.push(*g as i64);
        }
        let mut k = k;
        while k>0 {
            let n = heap.pop().unwrap();
            if n == 1 {
                heap.push(1);
                break;
            }
            let m = (n as f64).sqrt() as i64;
            heap.push(m);
            k -= 1;
        }
        heap.iter().sum()
    }
}
```
# 2024-12-11
[2779. Maximum Beauty of an Array After Applying Operation](https://leetcode.com/problems/maximum-beauty-of-an-array-after-applying-operation/)

I thought about a lot of complicated ways to solve this problem... In particular, a "subsequence" simply means choosing any amount of elements... This means we can sort the elements.

From there, we notice we can choose a range of elements from n-k to n+k. Putting it another way, from n to n+2k. 

```Rust
impl Solution {
    pub fn maximum_beauty(nums: Vec<i32>, k: i32) -> i32 {
        let mut nums = nums;
        nums.sort_unstable();
        let mut i = 0;
        let mut j = 0;
        let mut c = 0;
        while i<nums.len() && j<nums.len() {
            while j<nums.len() && nums[j]<=nums[i]+2*k {
                j += 1;
            }
            if j-i > c {
                c = j-i;
            }
            i += 1;
        }
        c as i32
    }
}
```

# 2024-12-10
[2981. Find Longest Special Substring That Occurs Thrice I](https://leetcode.com/problems/find-longest-special-substring-that-occurs-thrice-i/)

```Rust
use std::collections::HashMap;

impl Solution {
    pub fn maximum_length(s: String) -> i32 {
        let mut lo = 1;
        let mut hi = s.len();

        while lo<hi-1 {
            let m = (lo+hi)/2;
            if check(&s, m) {
                lo = m;
            } else {
                hi = m;
            }
        }

        if check(&s, lo) {
            return lo as i32;
        } else if check(&s, hi) {
            return hi as i32;
        } else {
            return -1;
        }
    }
}

fn check(s: &String, k: usize) -> bool {
    let mut map = HashMap::new();
    for w in s.as_bytes().windows(k) {
        let a = w[0];
        let mut b = false;
        for i in 1..w.len() {
            if a!=w[i] {
                b = true;
                break;
            }
        }
        if b {
            continue;
        }
        map.entry(w)
            .and_modify(|e| {*e += 1})
            .or_insert(1);
        if map[w] >= 3 {
            //println!("{:?}",String::from_utf8(w.to_vec()));
            return true;
        }
    }
    false
}
```

# 2024-12-09
[3152. Special Array II](https://leetcode.com/problems/special-array-ii/)

```Rust
impl Solution {
    pub fn is_array_special(nums: Vec<i32>, queries: Vec<Vec<i32>>) -> Vec<bool> {
        let mut a: Vec<i32> = vec![0].into_iter().chain(nums.windows(2).map(|x| (x[0]^x[1])&1)).collect();
        //println!("{:?}",a);
        let b: Vec<i32> = a.iter().scan(0, |state, &x| {
            *state = *state+x;
            Some(*state)
        }).collect();
        //println!("{:?}",b);

        queries.iter().map(|x| b[x[1] as usize]-b[x[0] as usize]==x[1]-x[0]).collect()
    }
}
```

# 2024-12-08

## TLE
```Rust
use std::collections::BinaryHeap;

impl Solution {
    pub fn max_two_events(events: Vec<Vec<i32>>) -> i32 {
        let mut best = 0;
        let mut heap = BinaryHeap::new();
        for e in events.iter() {
            heap.push((e[2],e[0],e[1]));
        }
        while !heap.is_empty() {
            let mut rest = BinaryHeap::new();
            while let Some(e) = heap.pop() {
                //println!("{:?} {:?}",e,heap);
                let mut sum = e.0;
                if let Some(f) = heap.peek() {
                    let mut best_possible = e.0 + f.0;
                    if best_possible<=best {
                        return best;
                    }
                }
                while let Some(f) = heap.pop() {
                    if e.2<f.1 || f.2<e.1 {
                        sum += f.0;
                        break;
                    }
                    rest.push(f);
                }
                if sum>best {
                    best = sum;
                }
                heap.append(&mut rest);
            }
        }
        best
    }
}
```
# 2024-12-07
[1760. Minimum Limit of Balls in a Bag](https://leetcode.com/problems/minimum-limit-of-balls-in-a-bag/)

Followed the hints for this one. You create a check function to see if it's possible to split the bags with k balls. If there's too many bags then it's not possible. Then you set up a binary search. It's really fast since it runs at O(log(n))...

## Binary Search
```Rust
impl Solution {
    pub fn minimum_size(nums: Vec<i32>, max_operations: i32) -> i32 {
        let mut lo = 1;
        let mut hi = *nums.iter().max().unwrap();

        while lo<hi-1 {
            let mi = (lo+hi)/2;
            if check(&nums, max_operations, mi) {
                hi = mi;
            } else {
                lo = mi;
            }
        }

        if check(&nums, max_operations, lo) {
            lo
        } else {
            hi
        }
    }
}

fn check(nums: &Vec<i32>, ops: i32, k: i32) -> bool {
    let mut n_bags = 0;
    for n in nums.iter() {
        n_bags += (n+k-1)/k;
    }
    n_bags <= nums.len() as i32 + ops
}
```

## TLE
```Rust
use std::collections::BinaryHeap;

impl Solution {
    pub fn minimum_size(nums: Vec<i32>, max_operations: i32) -> i32 {
        let mut ops = max_operations;
        let mut heap = BinaryHeap::new();
        
        for n in nums.iter() {
            heap.push((*n,*n,1));
        }

        while ops>0 {
            let (_,n,d) = heap.pop().unwrap();
            let v = (n+d)/(d+1);
            heap.push((v,n,d+1));
            ops-=1;
        }

        heap.pop().unwrap().0
    }
}
```

# 2024-12-06
[2554. Maximum Number of Integers to Choose From a Range I](https://leetcode.com/problems/maximum-number-of-integers-to-choose-from-a-range-i/)

Got it on second try. The banned list didn't guarantee no duplicates.

```Rust
impl Solution {
    pub fn max_count(banned: Vec<i32>, n: i32, max_sum: i32) -> i32 {
        let mut banned = banned;
        banned.sort_unstable();
        banned.dedup();
        //println!("{:?}", banned);
        let mut i = 1;
        let mut m = 0;
        let mut sum = 0;
        let mut count = 0;
        while i<=n && sum<max_sum {
            if m<banned.len() && i==banned[m] {
                m += 1;
            } else {
                //print!("{} ",i);
                sum += i;
                count += 1;
            }
            i += 1;
        }
        //println!(": {}",sum);
        if sum>max_sum {
            i -= 1;
            sum -= i;
            count -= 1;
        }
        //println!("{}",sum);
        count
    }
}
```

# 2024-12-05
[2337. Move Pieces to Obtain a String](https://leetcode.com/problems/move-pieces-to-obtain-a-string/)

Fairly straightforward. Strangely got it first try.

Anyways, you simply compare the ordering of L's and R's from both strings. Then check if the positions are valid or not.

```Rust
impl Solution {
    pub fn can_change(start: String, target: String) -> bool {
        let mut a = Vec::new();
        let mut b = Vec::new();

        for (i,c) in start.chars().enumerate() {
            match c {
                'L' => {
                    a.push((c,i));
                }
                'R' => {
                    a.push((c,i));
                }
                _ => ()
            }
        }
        for (i,c) in target.chars().enumerate() {
            match c {
                'L' => {
                    b.push((c,i));
                }
                'R' => {
                    b.push((c,i));
                }
                _ => ()
            }
        }

        if a.len() != b.len() {
            return false;
        }

        for i in 0..a.len() {
            if a[i].0 != b[i].0 {
                return false;
            }
            if a[i].0=='L' && a[i].1<b[i].1 {
                return false;
            }
            if a[i].0=='R' && a[i].1>b[i].1 {
                return false;
            }
        }

        true
    }
}
```

# 2024-12-04
[2825. Make String a Subsequence Using Cyclic Increments](https://leetcode.com/problems/make-string-a-subsequence-using-cyclic-increments/)

Playing around with iterators.

```Rust
impl Solution {
    pub fn can_make_subsequence(str1: String, str2: String) -> bool {
        let mut iter = str2.chars().peekable();

        for b in str1.chars() {
            match iter.peek() {
                None => {
                    return true;
                }
                Some(a) => {
                    let c = *a as i32 - b'a' as i32;
                    let d1 = b as i32 - b'a' as i32;
                    let d2 = (d1+1)%26;
                    if c==d1 || c==d2 {
                        iter.next();
                    }
                }
            }
        }

        match iter.next() {
            None => {
                true
            }
            Some(_) => {
                false
            }
        }
    }
}
```

# 2024-12-03
[2109. Adding Spaces to a String](https://leetcode.com/problems/adding-spaces-to-a-string/)

Not the fastest solution though.

```Rust
impl Solution {
    pub fn add_spaces(s: String, spaces: Vec<i32>) -> String {
        let mut out = String::new();
        let mut n = 0;
        for (i,c) in s.chars().enumerate() {
            if n<spaces.len() && spaces[n] == i as i32 {
                out.push(' ');
                n += 1;
            }
            out.push(c);
        }
        out
    }
}
```
# 2024-12-02
[1455. Check If a Word Occurs As a Prefix of Any Word in a Sentence](https://leetcode.com/problems/check-if-a-word-occurs-as-a-prefix-of-any-word-in-a-sentence/)

Just split the string into parts and compare. String comparison in Rust is easy... At least for this problem.

```Rust
impl Solution {
    pub fn is_prefix_of_word(sentence: String, search_word: String) -> i32 {
        for (i,w) in sentence.split(' ').enumerate() {
            //println!("{}",w);
            if w.len()>=search_word.len() && w[0..search_word.len()] == search_word {
                return (i+1) as i32;
            }
        }
        return -1;
    }
}
```
# 2024-12-01
[1346. Check If N and Its Double Exist](https://leetcode.com/problems/check-if-n-and-its-double-exist/)

Need to deal with a special case for 0 since 0 * 2 = 0.

```Rust
use std::collections::HashSet;

impl Solution {
    pub fn check_if_exist(arr: Vec<i32>) -> bool {
        let mut set = HashSet::new();
        let mut zeros = 0;
        for e in arr.iter() {
            if *e==0 {
                zeros += 1;
            } else {
                set.insert(e);
            }
        }
        if zeros>1 {
            return true;
        }
        for e in arr.iter() {
            if set.contains(&(e*2)) {
                return true;
            }
        }
        false
    }
}
```
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
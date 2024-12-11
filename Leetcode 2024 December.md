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
# 2024-11-20
[2516. Take K of Each Character From Left and Right](https://leetcode.com/problems/take-k-of-each-character-from-left-and-right/)

```Rust
impl Solution {
    pub fn take_characters(s: String, k: i32) -> i32 {
        if k==0 {
            return 0;
        }

        let mut counta = Vec::new();
        let mut countb = Vec::new();
        let mut countc = Vec::new();

        let mut a = 0;
        let mut b = 0;
        let mut c = 0;

        counta.push(a);
        countb.push(b);
        countc.push(c);

        for d in s.chars() {
            match d {
                'a' => {
                    a+=1;
                }
                'b' => {
                    b+=1;
                }
                'c' => {
                    c+=1;
                }
                _ => ()
            }
            counta.push(a);
            countb.push(b);
            countc.push(c);
        }

        if a<k || b<k || c<k {
            return -1;
        }

        let mut hi = s.len()- k as usize * 3;
        let mut lo = 1;
        //println!("{} {}",lo,hi);

        if hi==0 {
            return s.len() as i32;
        }

        while lo<hi-1 {
            let m = (lo+hi)/2;
            //println!("{} {} {}",lo,m,hi);
            if check(&counta,&countb,&countc,a-k,b-k,c-k,m) {
                lo = m;
            } else {
                hi = m;
            }
        }

        if check(&counta,&countb,&countc,a-k,b-k,c-k,hi) {
            (s.len()-hi) as i32
        } else {
            (s.len()-lo) as i32
        }
    }
}

fn check(ca: &Vec<i32>, cb: &Vec<i32>, cc: &Vec<i32>, ka: i32, kb: i32, kc: i32, m: usize) -> bool {
    if m == 0 {
        return true;
    }
    for i in 0..ca.len()-m {
        let a = ca[i+m]-ca[i];
        let b = cb[i+m]-cb[i];
        let c = cc[i+m]-cc[i];
        if a<=ka && b<=kb && c<=kc {
            return true;
        }
    }
    false
}
```

# 2024-11-19
[2461. Maximum Sum of Distinct Subarrays With Length K](https://leetcode.com/problems/maximum-sum-of-distinct-subarrays-with-length-k/)

Uses a HashMap and a window sum.

The window sum is typical for finding sums of a certain subarray length. While the HashMap is used to indicate distinct elements.

```Rust
use std::collections::HashMap;

impl Solution {
    pub fn maximum_subarray_sum(nums: Vec<i32>, k: i32) -> i64 {
        let mut max = 0;
        let mut sum = 0;
        let mut map = HashMap::new();
        let k = k as usize;
        for i in 0..k {
            map.entry(nums[i]).and_modify(|v| *v+=1).or_insert(1);
            sum += nums[i] as i64;
        }
        if map.len() == k {
            max = sum;
        }
        for i in 0..nums.len()-k {
            let v = map.remove(&nums[i]).unwrap();
            if v>1 {
                map.insert(nums[i],v-1);
            }
            sum -= nums[i] as i64;
            sum += nums[i+k] as i64;
            map.entry(nums[i+k]).and_modify(|v| *v+=1).or_insert(1);
            if map.len() == k && sum>max {
                max = sum;
            }
        }
        max
    }
}
```
# 2024-11-18
[1652. Defuse the Bomb](https://leetcode.com/problems/defuse-the-bomb/)

```Rust
impl Solution {
    pub fn decrypt(code: Vec<i32>, k: i32) -> Vec<i32> {
        if k==0 {
            let out = vec![0; code.len()];
            return out;
        }
        let m = if k>0 {
            k as usize
        } else {
            (-k) as usize
        };
        let mut a = if k>0 {
            [&code[1..code.len()],&code[0..k as usize]].concat()
        } else {
            [&code[code.len()-m..code.len()],&code[0..code.len()-1]].concat()
        };
        //println!("{:?}",a);
        a.windows(m).map(|x| x.iter().fold(0, |acc,v| acc+v)).collect()
    }
}
```

# 2024-11-17
[862. Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/)

## Attempt
```Rust
impl Solution {
    pub fn shortest_subarray(nums: Vec<i32>, k: i32) -> i32 {
        let k = k as i64;
        let mut cumsum = Vec::new();
        cumsum.push(0 as i64);
        for e in nums.iter() {
            let a = cumsum.last().unwrap() + *e as i64;
            cumsum.push(a);
        }
        println!("{:?}",cumsum);
        let mut cumsum2 = cumsum.clone();
        for i in (1..cumsum.len()).rev() {
            if nums[i-1]<0 && cumsum[i-1]>cumsum[i] {
                cumsum[i-1] = cumsum[i];
            }
        }
        println!("{:?}",cumsum);
        let mut lo=1;
        let mut hi=nums.len();
        while lo<hi-1 {
            let m = (lo+hi)/2;
            if check(&cumsum, k, m) {
                hi = m;
            } else {
                lo = m;
            }
        }
        println!("{} {}",lo,hi);
        if check(&cumsum, k, lo) {
            return lo as i32;
        } else if check(&cumsum, k, hi) {
            return hi as i32;
        }

        let mut cumsum = cumsum2;
        for i in 0..cumsum.len()-1 {
            if nums[i]<0 && cumsum[i]>cumsum[i+1] {
                cumsum[i+1] = cumsum[i];
            }
        }
        println!("{:?}",cumsum);
        let mut lo=1;
        let mut hi=nums.len();
        while lo<hi-1 {
            let m = (lo+hi)/2;
            if check(&cumsum, k, m) {
                hi = m;
            } else {
                lo = m;
            }
        }
        println!("{} {}",lo,hi);
        if check(&cumsum, k, lo) {
            return lo as i32;
        } else if check(&cumsum, k, hi) {
            return hi as i32;
        }

        -1
    }
}

fn check(cumsum: &Vec<i64>, k: i64, m: usize) -> bool {
    for i in 0..cumsum.len()-m {
        if cumsum[i+m]-cumsum[i] >= k {
            return true;
        }
    }
    false
}
```

# 2024-11-16
[3254. Find the Power of K-Size Subarrays I](https://leetcode.com/problems/find-the-power-of-k-size-subarrays-i/)

Thinking about it, I think the better solution is to count the number of consecutive terms then output whenever it equals or is greater than k-1.

## Counting
```Rust
impl Solution {
    pub fn results_array(nums: Vec<i32>, k: i32) -> Vec<i32> {
        if k==1 {
            return nums;
        }
        let k = k as usize;
        let mut out = Vec::new();
        let mut m = 0;
        for (i,w) in nums.windows(2).enumerate() {
            if w[0]+1==w[1] {
                m += 1;
            } else {
                m = 0;
            }
            if m>=k-1 {
                out.push(nums[i+1]);
            } else {
                out.push(-1);
            }
        }
        out[k-2..].to_vec()
    }
}
```

## Logical AND
```Rust
impl Solution {
    pub fn results_array(nums: Vec<i32>, k: i32) -> Vec<i32> {
        if k==1 {
            return nums;
        }
        let k = k as usize;
        let mut b = vec![false; nums.len()];
        for i in 1..nums.len() {
            if nums[i-1]+1==nums[i] {
                b[i] = true;
            }
        }
        //println!("{:?}",b);
        let mut out = vec![0; nums.len()-k+1];
        let mut i = 0;
        let mut iter = b.windows(k-1);
        iter.next();
        for w in iter {
            if w.iter().fold(true, |acc,x| acc && *x) {
                out[i] = nums[i+k-1];
            } else {
                out[i] = -1;
            }
            i += 1;
        }

        out
    }
}
```
# 2024-11-15
[1574. Shortest Subarray to be Removed to Make Array Sorted](https://leetcode.com/problems/shortest-subarray-to-be-removed-to-make-array-sorted/)

Binary search implementation. Mainly worked on edge cases until every test case passed. Not the most clean code as there are a bunch of "patches" done like adding one here or subtracting another one here... Indexing stuff...

```Rust
impl Solution {
    pub fn find_length_of_shortest_subarray(arr: Vec<i32>) -> i32 {
        let mut arr = arr;
        arr.insert(0,0);
        arr.push(1_000_000_000);
        let mut a = 0;
        while a<arr.len()-1 && arr[a]<=arr[a+1] {
            a += 1;
        }
        println!("{} {}",a,arr[a]);
        if a==arr.len()-1 {
            return 0;
        }
        let mut b = arr.len()-1;
        while b>0 && arr[b]>=arr[b-1] {
            b -= 1;
        }
        println!("{} {}",b,arr[b]);
        
        if arr[0]>arr[arr.len()-1] {
            return if a+1>arr.len()+1-b {
                arr.len()+1-a
            } else {
                b
            } as i32;
        }
        let mut lo = b-a;
        let mut hi = arr.len()-1;
        while lo<hi-1 {
            let m = (lo+hi)/2;
            if test(&arr,a,b,m) {
                hi = m;
            } else {
                lo = m;
            }
        }
        if test(&arr,a,b,lo) {
            (lo-1) as i32
        } else {
            (hi-1) as i32
        }
    }
}

fn test(arr: &Vec<i32>, a: usize, b: usize, k: usize) -> bool {
    for i in 0..=a {
        if i+k>=b && i+k<arr.len() && arr[i]<=arr[i+k] {
            return true;
        }
    }
    false
}
```
# 2024-11-14
[2064. Minimized Maximum of Products Distributed to Any Store](https://leetcode.com/problems/minimized-maximum-of-products-distributed-to-any-store/)

Took the hints for this one. Use the maximum of quantities as the upper bound for the binary search.

I still haven't figured out how to really write a binary search loop... It's not as "clean" as I want but it works. In this case, the result for comparison is true or false. A bad loop condition can lead to an infinite loop.

## Binary Search
```Rust
impl Solution {
    pub fn minimized_maximum(n: i32, quantities: Vec<i32>) -> i32 {
        let q = quantities;
        let mut top = *q.iter().reduce(|acc,x| if acc > x {acc} else {x}).unwrap();
        let mut bot = 1;
        while bot<top-1 {
            let k = (bot+top)/2;
            if can_dist(&q,n,k) {
                top = k;
            } else {
                bot = k;
            }
        }
        if can_dist(&q,n,bot) {
            bot
        } else {
            top
        }
    }
}

fn can_dist(q: &Vec<i32>, n: i32, k: i32) -> bool {
    let mut m = 0;
    for e in q.iter() {
        let a = (e+k-1)/k;
        m += a;
        if m>n {
            return false;
        }
    }
    true
}
```

## Greedy Attempt
```Rust
impl Solution {
    pub fn minimized_maximum(n: i32, quantities: Vec<i32>) -> i32 {
        let mut q = quantities.clone();
        q.sort_unstable();
        let mut sum = q.iter().fold(0, |acc,e| acc+e);
        let mut n = n;
        let mut max = 0;

        for e in q.iter() {
            let a = sum/n;
            let mut m = (e+a-1)/a;
            if m==0 {
                m = 1;
            }
            let d = e/m;
            print!("{}={}*{} ",e,m,d);
            if d>max {
                max = d;
            }
            sum -= e;
            n -= m;
        }

        max
    }
}
```
# 2024-11-13
[2563. Count the Number of Fair Pairs](https://leetcode.com/problems/count-the-number-of-fair-pairs/)

Works good enough. A single sort but a lot of array element removals... In theory O(n^2). But searches and calculations cost O(n * logn).

For the second version we have O(n * logn). We can double count as long as we divide by two at the end. The special cases are when the index is itself. (Read some of the comments to figure it out... We notice that we are matching every possible pair and the order doesn't matter since we're computing a sum.) 

## Best (Mathematical)
```Rust
impl Solution {
    pub fn count_fair_pairs(nums: Vec<i32>, lower: i32, upper: i32) -> i64 {
        let mut a = nums.clone();
        a.sort_unstable();
        let mut b = 0;
        for e in nums.iter() {
            let l = a.partition_point(|&x| x<lower-e);
            let u = a.partition_point(|&x| x<=upper-e);
            let mut d = u-l;
            if *e>=lower-e && *e<=upper-e {
                d -= 1;
            }
            if u>=l {
                b += d as i64;
            }
        }
        b/2
    }
}
```

## Bruteforce
```Rust
impl Solution {
    pub fn count_fair_pairs(nums: Vec<i32>, lower: i32, upper: i32) -> i64 {
        let mut a = nums.clone();
        a.sort_unstable();
        let mut b = 0;
        for e in nums.iter() {
            match a.binary_search(e) {
                Ok(i) => {
                    a.remove(i);
                }
                Err(_) => ()
            }
            let l = a.partition_point(|&x| x<lower-e);
            let u = a.partition_point(|&x| x<=upper-e);
            if u>=l {
                b += (u-l) as i64;
            }
        }
        b
    }
}
```

# 2024-11-12
[2070. Most Beautiful Item for Each Query](https://leetcode.com/problems/most-beautiful-item-for-each-query/)

Not the fastest solution but works. We sort then calculate the best "beauty" at each price point. From there each query is a binary search.

Using unstable sort makes it 100% fastest as well as use less memory. This is the best solution.

```Rust
impl Solution {
    pub fn maximum_beauty(items: Vec<Vec<i32>>, queries: Vec<i32>) -> Vec<i32> {
        let mut items = items;
        items.sort_unstable();
        for i in 0..items.len()-1 {
            if items[i+1][1]<items[i][1] {
                items[i+1][1] = items[i][1];
            }
        }
        for i in (0..items.len()-1).rev() {
            if items[i][0] == items[i+1][0] {
                items[i][1] = items[i+1][1];
            }
        }
        //items.dedup();
        //println!("{:?}",items);
        let mut out = vec![0; queries.len()];
        for i in 0..queries.len() {
            out[i] = match items.binary_search_by(|x| x[0].cmp(&queries[i])) {
                Ok(a) => {
                    items[a][1]
                }
                Err(a) => {
                    if a < 1 {
                        0
                    } else {
                        items[a-1][1]
                    }
                }
            }
        }
        out
    }
}
```

# 2024-11-11
[2601. Prime Subtraction Operation](https://leetcode.com/problems/prime-subtraction-operation/)

Took a greedy approach and start subtracting primes from the end of the array. The problem simply becomes an inductive problem.

It's always true if we can have negative numbers. Though a counter example shows we can't have 0. (5,8,3) can become (0,1,3).

```Rust
impl Solution {
    pub fn prime_sub_operation(nums: Vec<i32>) -> bool {
        let p = vec![0, 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31,
            37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83,
            89, 97, 101, 103, 107, 109, 113, 127, 131, 137,
            139, 149, 151, 157, 163, 167, 173, 179, 181, 191,
            193, 197, 199, 211, 223, 227, 229, 233, 239, 241,
            251, 257, 263, 269, 271, 277, 281, 283, 293, 307,
            311, 313, 317, 331, 337, 347, 349, 353, 359, 367,
            373, 379, 383, 389, 397, 401, 409, 419, 421, 431,
            433, 439, 443, 449, 457, 461, 463, 467, 479, 487,
            491, 499, 503, 509, 521, 523, 541, 547, 557, 563,
            569, 571, 577, 587, 593, 599, 601, 607, 613, 617,
            619, 631, 641, 643, 647, 653, 659, 661, 673, 677,
            683, 691, 701, 709, 719, 727, 733, 739, 743, 751,
            757, 761, 769, 773, 787, 797, 809, 811, 821, 823,
            827, 829, 839, 853, 857, 859, 863, 877, 881, 883,
            887, 907, 911, 919, 929, 937, 941, 947, 953, 967,
            971, 977, 983, 991, 997, 1009];
        let mut nums = nums;
        for i in (0..nums.len()-1).rev() {
            let mut j=0;
            while nums[i]-p[j]>=nums[i+1] {
                j+=1;
            }
            nums[i] -= p[j];
            if nums[i] < 1 {
                return false;
            }
        }
        true
    }
}
```

# 2024-11-10
[3097. Shortest Subarray With OR at Least K II](https://leetcode.com/problems/shortest-subarray-with-or-at-least-k-ii/)

Quite a doozy for logic. There are several ideas that need to be proved.

The main idea is having left and right bounds. Starting the the left bound at 0, we calculate the subarray cumulative OR until we have a value over or equal to k. We then recalculate the subarray from the right bound to find our new left bound. These left and right bounds represent the smallest (local) subarray which have a value over or equal to k. We can prove it's the smallest via contradiction.

In short, if we suppose there's a smaller subarray within, we should expect those to be our left and right bounds instead. However during calculation, we never get a value over k. Thus the smaller subarray must not exist.

From there, we simply repeat by incrementing the left bound by 1. (Then find the right bound, the new left bound, increment by 1, repeat...) We do this until we hit the end. Or we calculate a value lower despite hitting the end. (We OR everything in the leftover subarray.)

Sounds simple but the logic is meandering. Took a while for all of the pieces to fall into place. Mainly focused on not getting TLE with an O(n^2) solution. This solution seems like O(n^1.5) in terms of how many OR operations are done. The refinement method helps to remove a lot of candidates/computation.

To reiterate:
- Left bound = 0
- Calculate and find right bound
- Calculate and  refine left bound
- Record subarray length
- Increment left bound by 1
- Repeat and find next right bound...
- Return shortest bounds

## Logic
```Rust
impl Solution {
    pub fn minimum_subarray_length(nums: Vec<i32>, k: i32) -> i32 {
        if k==0 {
            return 1;
        }

        let mut l = 0;
        let mut best = nums.len()+1;

        while l<nums.len() {
            let mut i = l;
            let mut a = 0;
            while i<nums.len() && a<k {
                a |= nums[i];
                i+=1;
            }
            if a<k {
                break;
            }
            i -= 1;
            let r = i;
            a = 0;
            while i<nums.len() && a<k {
                a |= nums[i];
                i-=1;
            }
            i += 1;
            l = i;
            if r-l+1 < best {
                best = r-l+1;
            }
            //println!("{} {}",l,r);
            l += 1;
        }

        if best <= nums.len() {
            best as i32
        } else {
            -1
        }
    }
}
```

## Doesn't work
```Rust
impl Solution {
    pub fn minimum_subarray_length(nums: Vec<i32>, k: i32) -> i32 {
        if k==0 {
            return 1;
        }
        let mut t = 0;
        for i in 0..nums.len() {
            t |= nums[i];
        }
        if t<k {
            return -1;
        }
        let mut l = 0;
        let mut r = 0;
        let mut best = nums.len();
        while r<nums.len() {
            let mut a = 0;
            while r<nums.len() && a<k {
                a |= nums[r];
                r += 1;
            }
            l = r-1;
            a = 0;
            while l<nums.len() && a<k {
                a |= nums[l];
                l -= 1;
            }
            l += 1;
            if r-l < best {
                best = r-l;
            }
            //println!("{} {}",l,r);
        }
        best as i32
    }
}
```

## TLE
```Rust
impl Solution {
    pub fn minimum_subarray_length(nums: Vec<i32>, k: i32) -> i32 {
        let mut b = 0;
        for i in 0..nums.len() {
            b |= nums[i];
        }
        if b<k {
            return -1;
        }

        let mut cum = vec![0; nums.len()];
        let mut best = nums.len();
        for i in 0..nums.len() {
            let mut j = i;
            let mut a = nums[j];
            while j<nums.len() && a<k {
                a |= cum[j];
                cum[j] = a;
                j -= 1;
            }
            if a>=k && j<i {
                j += 1;
            }
            if j>nums.len() {
                j += 1;
            }
            if a>=k && i-j+1<best {
                best = i-j+1;
                //print!("*");
            }
            //println!("{:?} {} {}",cum,i,j);
        }
        best as i32
    }
}
```
# 2024-11-09
[3133. Minimum Array End](https://leetcode.com/problems/minimum-array-end/)

An interesting "operator". We're basically counting numbers that share the same AND value as the initial number.

An extreme example would show the pattern well... Say x=1024 and n=123. The answer would simply be 1024+123-1 = 1146. 1024 has the 10th bit set. Thus if n is less than x=1024, we simply add and get the answer. An n less than 1024 will not share any bits with x=1024.

Essentially, we just add the two numbers while skipping over the bits that are set.

```Rust
impl Solution {
    pub fn min_end(n: i32, x: i32) -> i64 {
        let mut n = (n-1) as i64;
        let mut x = x as i64;
        let mut b = 1 as i64;
        while n>0 {
            while b&x>0 {
                b = b<<1;
            }
            if n&1 > 0 {
                x |= b;
            }
            b <<= 1;
            n >>= 1;
        }
        x
    }
}
```
# 2024-11-08
[1829. Maximum XOR for Each Query](https://leetcode.com/problems/maximum-xor-for-each-query/)

XOR does have a particular logic to it.

The maximum value we can get is 2^maximumBit - 1.

So to maximize, we have:
```
k XOR nums[0] XOR nums[1] XOR ... XOR nums[i] = 2^maximumBit - 1
```
For each i.
Then using XOR algebra we can solve for k:
```
k XOR nums[0] XOR nums[1] XOR ... XOR nums[i] (XOR k) = 2^maximumBit - 1 (XOR k)
nums[0] XOR nums[1] XOR ... XOR nums[i] (XOR 2^maximumBit) = 2^maximumBit - 1 XOR k (XOR 2^maximumBit)
k = nums[0] XOR nums[1] XOR ... XOR nums[i] XOR 2^maximumBit
```


```Rust
impl Solution {
    pub fn get_maximum_xor(nums: Vec<i32>, maximum_bit: i32) -> Vec<i32> {
        let mut out = vec![0; nums.len()];
        let mut a = (1<<maximum_bit)-1;
        for i in 0..nums.len() {
            a ^= nums[i];
            out[i] = a;
        }
        out.reverse();
        out
    }
}
```

# 2024-11-07
[2275. Largest Combination With Bitwise AND Greater Than Zero](https://leetcode.com/problems/largest-combination-with-bitwise-and-greater-than-zero/)

Just counting. More specifically, we AND with only a single bit for every bit. Using "greedy" logic, having only one bit set will be sufficient for getting the largest combination.

```Rust
impl Solution {
    pub fn largest_combination(candidates: Vec<i32>) -> i32 {
        let mut a = 1;
        let mut count = vec![0; 24];
        for i in 0..24 {
            for c in candidates.iter() {
                if c&a != 0 {
                    count[i] += 1;
                }
            }
            a <<= 1;
        }
        *count.iter().max().unwrap()
    }
}
```

# 2024-11-06
[3011. Find if Array Can Be Sorted](https://leetcode.com/problems/find-if-array-can-be-sorted/)

Tried to be lazy with it... It has to be done the "proper" way.

## Proper Method
```Rust
impl Solution {
    pub fn can_sort_array(nums: Vec<i32>) -> bool {
        let mut b = Vec::new();
        for s in nums.chunk_by(|x,y| x.count_ones()==y.count_ones()) {
            let mut t = s.to_vec();
            t.sort_unstable();
            b.append(&mut t);
        }
        //println!("{:?}",b);
        for w in b.windows(2) {
            if w[0] > w[1] {
                return false;
            }
        }
        true
    }
}
```

## Lazy (Doesn't work)
```Rust
impl Solution {
    pub fn can_sort_array(nums: Vec<i32>) -> bool {
        let count1: Vec<_> = nums.iter().map(|x| x.count_ones()).collect();
        let mut nums = nums;
        nums.sort_unstable();
        let count2: Vec<_> = nums.iter().map(|x| x.count_ones()).collect();
        
        count1 == count2
    }
}
```

# 2024-11-05
[2914. Minimum Number of Changes to Make Binary String Beautiful](https://leetcode.com/problems/minimum-number-of-changes-to-make-binary-string-beautiful/)

Just reading the problem carefully. I'm really digging Rust's idioms.

```Rust
impl Solution {
    pub fn min_changes(s: String) -> i32 {
        let mut out = 0;
        for w in s.as_bytes().chunks(2) {
            if w[0]!=w[1] {
                out += 1;
            }
        }
        out
    }
}
```

# 2024-11-04
[3163. String Compression III](https://leetcode.com/problems/string-compression-iii/)

Messy but works.

```Rust
impl Solution {
    pub fn compressed_string(word: String) -> String {
        let mut word = word;
        word.push('A');
        let mut out = String::new();

        let mut count = 1;
        let mut chara = word.chars().next().unwrap();

        for w in word.as_bytes().windows(2) {
            if w[0]==w[1] {
                count += 1;
            } else {
                while count > 9 {
                    out.push('9');
                    out.push(chara);
                    count -= 9;
                }
                out.push((count as u8 + b'0') as char);
                out.push(chara);
                chara = w[1] as char;
                count = 1;
            }
        }

        out
    }
}
```

# 2024-11-03
[796. Rotate String](https://leetcode.com/problems/rotate-string/)

A straight forward approach.

```Rust
impl Solution {
    pub fn rotate_string(s: String, goal: String) -> bool {
        if s.len() != goal.len() {
            return false;
        }

        let s = s.as_bytes();
        let goal = goal.as_bytes();

        for i in 0..s.len() {
            let mut a = 0;
            let mut b = i; 
            while b<s.len() && s[a] == goal[b] {
                a += 1;
                b += 1;
            }
            if b<s.len() {
                continue;
            }
            b = 0;
            while a<s.len() && s[a] == goal[b] {
                a += 1;
                b += 1;
            }
            if a<s.len() {
                continue;
            }
            return true;
        }

        false
    }
}
```
# 2024-11-02
[2490. Circular Sentence](https://leetcode.com/problems/circular-sentence/)

Nothing too crazy here. Just keeping it simple rather than trying to split the string apart into words.

```Rust
impl Solution {
    pub fn is_circular_sentence(sentence: String) -> bool {
        let s = sentence.into_bytes();
        if s[0] != *s.last().unwrap() {
            return false;
        }
        for w in s.windows(3) {
            if w[1] == b' ' && w[0] != w[2] {
                return false;
            }
        }
        true
    }
}
```

# 2024-11-01
[1957. Delete Characters to Make Fancy String](https://leetcode.com/problems/delete-characters-to-make-fancy-string/)

It's easier to just construct a new string.

```Rust
impl Solution {
    pub fn make_fancy_string(s: String) -> String {
        let mut s = s;
        s.push('A');
        s.push('A');
        let s = s.as_bytes();
        let mut t = String::new();
        for w in s.windows(3) {
            if w[0]==w[1] && w[1]==w[2] {
               continue; 
            } else {
                t.push(w[0] as char);
            }
        }
        t
    }
}
```
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
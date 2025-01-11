# 2025-01-11
[1400. Construct K Palindrome Strings](https://leetcode.com/problems/construct-k-palindrome-strings/)

A logic problem. Only two checks are needed.

```Rust
impl Solution {
    pub fn can_construct(s: String, k: i32) -> bool {
        if s.len() < k as usize {
            return false;
        }
        let mut count = vec![0; 26];
        for c in s.bytes() {
            count[(c-b'a') as usize] += 1;
        }
        let mut odds = 0;
        //let mut pairs = 0;
        for e in count.iter() {
            //pairs += e/2;
            if e&1 == 1 {
                odds += 1;
            }
        }
        if odds > k {
            return false;
        }
        true
    }
}
```

# 2025-01-10
[916. Word Subsets](https://leetcode.com/problems/word-subsets/)

We count the minimum amount of letters needed in words2 then we count the letters in words1 to compare against.

```Rust
impl Solution {
    pub fn word_subsets(words1: Vec<String>, words2: Vec<String>) -> Vec<String> {
        let mut count = vec![0; 26];
        let mut temp = vec![0; 26];
        for w in words2.iter() {
            for c in w.bytes() {
                temp[(c-b'a') as usize] += 1;
            }
            for i in 0..26 {
                if count[i]<temp[i] {
                    count[i] = temp[i];
                }
            }
            temp.fill(0);
        }
        //println!("{:?}",count);
        words1.into_iter().filter(|w| {
            for c in w.bytes() {
                temp[(c-b'a') as usize] += 1;
            }
            let mut pass = true;
            for i in 0..26 {
                if temp[i]<count[i] {
                    pass = false;
                    break;
                }
            }
            temp.fill(0);
            pass
        }).collect()
    }
}
```

# 2025-01-09
[2185. Counting Words With a Given Prefix](https://leetcode.com/problems/counting-words-with-a-given-prefix/)

## One-liner
```Rust
impl Solution {
    pub fn prefix_count(words: Vec<String>, pref: String) -> i32 {
        words.iter().filter(|w| w.starts_with(&pref)).count() as i32
    }
}
```

## Typical
```Rust
impl Solution {
    pub fn prefix_count(words: Vec<String>, pref: String) -> i32 {
        let mut count = 0;
        for w in words.iter() {
            if w.starts_with(&pref) {
                count += 1;
            }
        }
        count
    }
}
```

# 2025-01-08
[3042. Count Prefix and Suffix Pairs I](https://leetcode.com/problems/count-prefix-and-suffix-pairs-i/)

## Bruteforce
```Rust
impl Solution {
    pub fn count_prefix_suffix_pairs(words: Vec<String>) -> i32 {
        let mut count = 0;
        for i in 0..words.len() {
            for j in i+1..words.len() {
                let a = &words[i];
                let b = &words[j];
                if b.starts_with(a) && b.ends_with(a) {
                    count += 1;
                }
            }
        }
        count
    }
}
```

# 2025-01-07
[1408. String Matching in an Array](https://leetcode.com/problems/string-matching-in-an-array/)

Just bruteforce.

```Rust
impl Solution {
    pub fn string_matching(words: Vec<String>) -> Vec<String> {
        let mut out = Vec::new();
        let mut has = vec![false; words.len()];
        for i in 0..words.len() {
            for j in i+1..words.len() {
                if !has[j] && words[i].contains(&words[j]) {
                    //out.push(words[j].clone());
                    has[j] = true;
                }
                if !has[i] && words[j].contains(&words[i]) {
                    //out.push(words[i].clone());
                    has[i] = true;
                }
                
            }
        }
        for i in 0..words.len() {
            if has[i] {
                out.push(words[i].clone());
            }
        }
        out
    }
}
```

# 2025-01-06
[1769. Minimum Number of Operations to Move All Balls to Each Box](https://leetcode.com/problems/minimum-number-of-operations-to-move-all-balls-to-each-box/)

## Cleaner
```Rust
impl Solution {
    pub fn min_operations(boxes: String) -> Vec<i32> {
        let mut r = 0;
        let mut l = 0;
        let mut sum = 0;
        for (i,b) in boxes.chars().enumerate() {
            if b=='1' {
                r += 1;
                sum += i as i32;
            }
        }
        let mut out = vec![0; boxes.len()];
        for i in 0..boxes.len() {
            out[i] = sum;
            if boxes.as_bytes()[i]==b'1' {
                r -= 1;
                sum -= r;
                l += 1;
                sum += l;
            } else {
                sum -= r;
                sum += l;
            }
        }
        out
    }
}
```

## Fast
```Rust
impl Solution {
    pub fn min_operations(boxes: String) -> Vec<i32> {
        let mut boxes: Vec<_> = boxes.bytes().collect();
        for b in boxes.iter_mut() {
            *b -= b'0';
        }
        let mut r = 0;
        for b in boxes.iter() {
            r += *b as i32;
        }
        let mut l = 0;
        let mut sum = 0;
        for (i,b) in boxes.iter().enumerate() {
            if *b==1 {
                sum += i as i32;
            }
        }
        let mut out = vec![0; boxes.len()];
        for i in 0..boxes.len() {
            //println!("{} {} {}",l,r,sum);
            out[i] = sum;
            if boxes[i]==1 {
                r -= 1;
                sum -= r;
                l += 1;
                sum += l;
            } else {
                sum -= r;
                sum += l;
            }
        }
        out
    }
}
```

# 2025-01-05
[2381. Shifting Letters II](https://leetcode.com/problems/shifting-letters-ii/)

Hard problem for me... But it works. There is a certain trick to it...
## Works
```Rust
use std::collections::BinaryHeap;
use core::cmp::Reverse;

impl Solution {
    pub fn shifting_letters(s: String, shifts: Vec<Vec<i32>>) -> String {
        let mut heapa = BinaryHeap::new();
        for e in shifts.iter() {
            if e[2]==0 {
                heapa.push(Reverse((e[0],e[1],-1)));
            } else {
                heapa.push(Reverse((e[0],e[1],1)));
            }
        }
        //println!("{:?}",heapa);
        let mut heapb: BinaryHeap<Reverse<(i32,i32)>>= BinaryHeap::new();
        let mut acc = 0;
        let mut sh = vec![0; s.len()];
        for i in 0..s.len() {
            while let Some(a) = heapa.peek() {
                if a.0.0<=i as i32 {
                    acc += a.0.2;
                    heapb.push(Reverse((a.0.1,a.0.2)));
                    heapa.pop();
                } else {
                    break;
                }
            }
            while let Some(b) = heapb.peek() {
                if b.0.0<i as i32 {
                    acc -= b.0.1;
                    heapb.pop();
                } else {
                    break;
                }
            }
            sh[i] = acc;
        }
        //println!("{:?}",heapa);
        //println!("{:?}",heapb);
        //println!("{:?}",sh);
        for i in 0..s.len() {
            sh[i] += (s.as_bytes()[i] - b'a') as i32;
            sh[i] %= 26;
            if sh[i]<0 {
                sh[i] += 26;
            }
            sh[i] += b'a' as i32;
        }
        String::from_utf8(sh.iter().map(|x| *x as u8).collect()).unwrap()
    }
}
```

## Draft 2
```Rust
use std::collections::BinaryHeap;
use core::cmp::Reverse;

impl Solution {
    pub fn shifting_letters(s: String, shifts: Vec<Vec<i32>>) -> String {
        let mut heapa = BinaryHeap::new();
        for e in shifts.iter() {
            if e[2]==0 {
                heapa.push(Reverse((e[0],e[1],-1)));
            } else {
                heapa.push(Reverse((e[0],e[1],1)));
            }
        }
        println!("{:?}",heapa);
        let mut heapb: BinaryHeap<Reverse<(i32,i32)>>= BinaryHeap::new();
        let mut acc = 0;
        let mut ind = 0;
        let mut sh = vec![0; s.len()];
        while let Some(a) = heapa.pop() {
            println!("{:?}",heapb);
            for i in ind..=a.0.0 {
                sh[i as usize] = acc;
                while let Some(b) = heapb.peek() {
                    if b.0.0 <= i {
                        acc += b.0.1;
                        heapb.pop();
                    } else {
                        break;
                    }
                }
            }
            ind = a.0.0;
            heapb.push(Reverse((a.0.1,a.0.2)));
        }
        println!("{:?}",heapb);
        println!("{:?}",sh);

        s
    }
}
```

## Draft 1
```Rust
use std::collections::BinaryHeap;
use core::cmp::Reverse;

impl Solution {
    pub fn shifting_letters(s: String, shifts: Vec<Vec<i32>>) -> String {
        let mut heapa = BinaryHeap::new();
        for e in shifts.iter() {
            if e[2]==0 {
                heapa.push(Reverse((e[0],e[1],-1)));
            } else {
                heapa.push(Reverse((e[0],e[1],1)));
            }
        }
        println!("{:?}",heapa);
        let mut heapb: BinaryHeap<Reverse<(i32,i32)>>= BinaryHeap::new();
        let mut acc = 0;
        let mut ind = 0;
        let mut sh = vec![0; s.len()];
        while let Some(a) = heapa.pop() {
            println!("{:?}, {}",a,ind);
            while let Some(b) = heapb.peek() {
                if b.0.0 <= ind {
                    heapb.pop();
                } else {
                    break;
                }
            }
            match heapb.peek() {
                None => {
                    ind = a.0.0;
                    acc += a.0.2;
                    heapb.push(Reverse((a.0.1,a.0.2)));
                }
                Some(b) => {
                    for i in ind..a.0.0 {
                        sh[i as usize] = acc;
                    }
                    ind = a.0.0;
                    acc += a.0.2;
                    heapb.push(Reverse((a.0.1,a.0.2)));
                }
            }
        }
        println!("{:?}",sh);

        s
    }
}
```

## Naive
```Rust
impl Solution {
    pub fn shifting_letters(s: String, shifts: Vec<Vec<i32>>) -> String {
        let mut a = vec![0; s.len()];
        for e in shifts.iter() {
            let b = e[0] as usize;
            let c = e[1] as usize;
            if e[2] == 0 {
                for i in b..=c {
                    a[i] += 25;
                }
            } else {
                for i in b..=c {
                    a[i] += 1;
                }
            }
        }
        println!("{:?}",a);
        let mut d = vec![0; s.len()];
        for i in 0..a.len() {
            d[i] = (((s.as_bytes()[i] - b'a') as i32 + a[i])%26) as u8 + b'a';
        }
        println!("{:?}",d);
        let out = String::from_utf8(d).unwrap();
        out
    }
}
```

# 2025-01-04
[1930. Unique Length-3 Palindromic Subsequences](https://leetcode.com/problems/unique-length-3-palindromic-subsequences/)

Since we're only considering a subsequence of 3 characters, we can really optimize our calculation. Runs in linear time and takes around 10ms. While the submitted solutions take around 133ms and uses a hash set...

The alternative is to keep a count for each index of the string... But the amount of space would be quite large (26 * s.len())... Instead, this solution opts to simply compute it instead. Another alternative is we only keep track of the counts for the first and last indexes for each character (26 * 26 * 2).

The submitted code (for Rust) is quite horrible... Using magic numbers like 97 or 123 to represent 'a' and 'z'... But on the other hand using a HashSet to check which characters are seen... Honestly garbage code. I assume a lot of people are simply cheating by copying the (bad) solution...

```Rust
impl Solution {
    pub fn count_palindromic_subsequence(s: String) -> i32 {
        let mut first = vec![usize::MAX; 26];
        let mut last = vec![usize::MAX; 26];
        let mut totals = vec![0; 26];
        for (i,c) in s.bytes().enumerate() {
            let c = (c-b'a') as usize;
            totals[c] += 1;
            if first[c] == usize::MAX {
                first[c] = i;
            }
            last[c] = i;
        }
        //println!("{:?}",first);
        //println!("{:?}",last);
        //println!("{:?}",totals);
        let mut exist = vec![false; 26];
        let mut sum = 0;
        for c in 0..26 {
            if totals[c] >= 2 {
                for i in first[c]+1..last[c] {
                    let d = (s.as_bytes()[i]-b'a') as usize;
                    exist[d] = true;
                }
                for b in exist.iter() {
                    if *b {
                        sum += 1;
                    }
                }
                for e in exist.iter_mut() {
                    *e = false;
                }
            }
        }
        sum
    }
}
```

# 2025-01-03
[2270. Number of Ways to Split Array](https://leetcode.com/problems/number-of-ways-to-split-array/)

You have to iterate at least twice. First to get the sum. Second to compare.

## Shorter
```Rust
impl Solution {
    pub fn ways_to_split_array(nums: Vec<i32>) -> i32 {
        let total = nums.iter().map(|x| *x as i64).sum();
        let mut sum = 0;
        let mut out = 0;
        for i in 0..nums.len()-1 {
            sum += nums[i] as i64;
            if sum*2>=total {
                out += 1;
            }
        }
        out
    }
}
```

## Longer
```Rust
impl Solution {
    pub fn ways_to_split_array(nums: Vec<i32>) -> i32 {
        if nums.len() <= 20000 {
            let mut sums = vec![0; nums.len()];
            let mut sum = 0;
            for i in 0..nums.len() {
                sum += nums[i];
                sums[i] = sum;
            }
            let total = sums[nums.len()-1];
            let mut out = 0;
            for i in 0..nums.len()-1 {
                if sums[i]*2>=total {
                    out += 1;
                }
            }
            out
        } else {
            let mut sums = vec![0; nums.len()];
            let mut sum = 0 as i64;
            for i in 0..nums.len() {
                sum += nums[i] as i64;
                sums[i] = sum;
            }
            let total = sums[nums.len()-1];
            let mut out = 0;
            for i in 0..nums.len()-1 {
                if sums[i]*2>=total {
                    out += 1;
                }
            }
            out
        }
    }
}
```

# 2025-01-02
[2559. Count Vowel Strings in Ranges](https://leetcode.com/problems/count-vowel-strings-in-ranges/)

I should use hash sets more often if I need to match multiple values... Well, a LUT is also a sort of hash set...

## Boolean Array
```Rust
impl Solution {
    pub fn vowel_strings(words: Vec<String>, queries: Vec<Vec<i32>>) -> Vec<i32> {
        let mut count = Vec::new();
        count.push(0);
        let mut c = 0;
        let mut set = vec![false; 26];
        set[(b'a'-b'a') as usize] = true;
        set[(b'e'-b'a') as usize] = true;
        set[(b'i'-b'a') as usize] = true;
        set[(b'o'-b'a') as usize] = true;
        set[(b'u'-b'a') as usize] = true;
        for w in words.iter() {
            let a = (w.as_bytes()[0]-b'a') as usize;
            let b = (w.as_bytes()[w.len()-1]-b'a') as usize;
            if set[a] && set[b] {
                c += 1;
            }
            count.push(c);
        }
        let mut out = Vec::new();
        for q in queries.iter() {
            out.push(count[(q[1]+1) as usize]-count[q[0] as usize]);
        }
        out
    }
}
```

## Hash Set Method
```Rust
use std::collections::HashSet;

impl Solution {
    pub fn vowel_strings(words: Vec<String>, queries: Vec<Vec<i32>>) -> Vec<i32> {
        let mut count = Vec::new();
        count.push(0);
        let mut c = 0;
        let mut set = HashSet::new();
        set.insert(b'a');
        set.insert(b'e');
        set.insert(b'i');
        set.insert(b'o');
        set.insert(b'u');
        for w in words.iter() {
            let a = w.as_bytes()[0];
            let b = w.as_bytes()[w.len()-1];
            if set.contains(&a) && set.contains(&b) {
                c += 1;
            }
            count.push(c);
        }
        let mut out = Vec::new();
        for q in queries.iter() {
            out.push(count[(q[1]+1) as usize]-count[q[0] as usize]);
        }
        out
    }
}
```

## Comparison Method
```Rust
impl Solution {
    pub fn vowel_strings(words: Vec<String>, queries: Vec<Vec<i32>>) -> Vec<i32> {
        let mut count = Vec::new();
        count.push(0);
        let mut c = 0;
        for w in words.iter() {
            let a = w.as_bytes()[0];
            let b = w.as_bytes()[w.len()-1];
            match a {
                b'a' | b'e' | b'i' | b'o' | b'u' => {
                    match b {
                        b'a' | b'e' | b'i' | b'o' | b'u' => {
                            c += 1;
                        }
                        _ => ()
                    }
                }
                _ => ()
            }
            count.push(c);
        }
        let mut out = Vec::new();
        for q in queries.iter() {
            out.push(count[(q[1]+1) as usize]-count[q[0] as usize]);
        }
        out
    }
}
```

# 2025-01-01
[1422. Maximum Score After Splitting a String](https://leetcode.com/problems/maximum-score-after-splitting-a-string/)

```Rust
impl Solution {
    pub fn max_score(s: String) -> i32 {
        let mut ones = 0;
        //let mut zeros = 0;
        for a in s.chars() {
            match a {
                '0' => (),//zeros+=1,
                '1' => ones+=1,
                _ => ()
            }
        }
        let mut ones2 = 0;
        let mut zeros2 = 0;
        let mut max = 0;
        for (i,a) in s.chars().enumerate() {
            if i==s.len()-1 {
                continue;
            }
            match a {
                '0' => zeros2+=1,
                '1' => ones2+=1,
                _ => ()
            }
            let b = zeros2+ones-ones2;
            //print!("{} ",b);
            if b>max {
                max = b;
            }
        }
        max
    }
}
```
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
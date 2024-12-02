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
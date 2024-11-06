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
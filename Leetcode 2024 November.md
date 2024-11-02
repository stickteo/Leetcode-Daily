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
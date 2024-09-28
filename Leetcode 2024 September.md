# 2024-09-28
[641. Design Circular Deque](https://leetcode.com/problems/design-circular-deque/)

A fairly standard implementation of a deque using a circular buffer. With most implementations, the devil are in the details...

A deque (a double ended queue) allows insertion in the front and back. A "single ended queue" would be a "typical" "vector". A vector only allows insertion at the back, while inserting in the front would be expensive.

Data is simply an array. We initialize with the exact size we need.

Front and back are indexes. For this implementation, we set the back to size-1. The front and back will always point to data when we call the "get" functions. An alternative design choice would be having the front and back point to empty spaces...

Len keeps track of the current "length" of the deque. Which is the current size. We need the length to indicate whether the deque is full or empty; We can simply check against zero or size. In theory, we can omit the use of length but we will lose one slot for the data; We would need an empty slot in data otherwise an empty deque is indistinguishable from a full deque... To allow the right size, we would need to allocate one more slot anyways... But that empty slot will not be used for anything useful... Therefore, the better design choice is just to have len.

An alternative implementation for a deque seems to be a doubly-linked list... But linked lists are generally horrible data structures (in my honest opinion). I don't really want to give a diatribe about how bad linked lists are... In short though, linked lists are slow, error-prone, and actually complex to implement properly.

```C
typedef struct {
    int *data;
    int size;
    int front;
    int back;
    int len;
} MyCircularDeque;

MyCircularDeque* myCircularDequeCreate(int k) {
    MyCircularDeque *deque = calloc(1,sizeof(MyCircularDeque));
    deque->data = malloc(k*sizeof(int));
    deque->size = k;
    deque->front = 0;
    deque->back = k-1;
    deque->len = 0;
    return deque;
}

bool myCircularDequeInsertFront(MyCircularDeque* obj, int value) {
    if (obj->len<obj->size) {
        obj->front = (obj->front+obj->size-1)%obj->size;
        obj->data[obj->front] = value;
        obj->len += 1;
        return true;
    } else {
        return false;
    }
}

bool myCircularDequeInsertLast(MyCircularDeque* obj, int value) {
    if (obj->len<obj->size) {
        obj->back = (obj->back+1)%obj->size;
        obj->data[obj->back] = value;
        obj->len += 1;
        return true;
    } else {
        return false;
    }
}

bool myCircularDequeDeleteFront(MyCircularDeque* obj) {
    if (obj->len>0) {
        obj->front = (obj->front+1)%obj->size;
        obj->len -= 1;
        return true;
    } else {
        return false;
    }
}

bool myCircularDequeDeleteLast(MyCircularDeque* obj) {
    if (obj->len>0) {
        obj->back = (obj->back+obj->size-1)%obj->size;
        obj->len -= 1;
        return true;
    } else {
        return false;
    }
}

int myCircularDequeGetFront(MyCircularDeque* obj) {
    if (obj->len>0) {
        return obj->data[obj->front];
    } else {
        return -1;
    }
}

int myCircularDequeGetRear(MyCircularDeque* obj) {
    if (obj->len>0) {
        return obj->data[obj->back];
    } else {
        return -1;
    }
}

bool myCircularDequeIsEmpty(MyCircularDeque* obj) {
    return obj->len == 0;
}

bool myCircularDequeIsFull(MyCircularDeque* obj) {
    return obj->len == obj->size;
}

void myCircularDequeFree(MyCircularDeque* obj) {
    free(obj->data);
    free(obj);
}

/**
 * Your MyCircularDeque struct will be instantiated and called as such:
 * MyCircularDeque* obj = myCircularDequeCreate(k);
 * bool param_1 = myCircularDequeInsertFront(obj, value);
 
 * bool param_2 = myCircularDequeInsertLast(obj, value);
 
 * bool param_3 = myCircularDequeDeleteFront(obj);
 
 * bool param_4 = myCircularDequeDeleteLast(obj);
 
 * int param_5 = myCircularDequeGetFront(obj);
 
 * int param_6 = myCircularDequeGetRear(obj);
 
 * bool param_7 = myCircularDequeIsEmpty(obj);
 
 * bool param_8 = myCircularDequeIsFull(obj);
 
 * myCircularDequeFree(obj);
*/
```

# 2024-09-26
[729. My Calendar I](https://leetcode.com/problems/my-calendar-i/)

I was able to get a 100% time run using a VecDeque. (A VecDeque was slightly faster than a Vec.)

The implementation is "straightforward". We simply store the pairs directly into a vector. Since the times come in pairs, a time at an even index will always be a start time while a time at an odd index will always be an end time. Furthermore, the pairs will always remain together since if we insert a pair that has intersecting time, the times will conflict.

For the specifics, Rust's "binary_search" allows us to search for a certain value within the vector. If a value exists, it returns an "Ok" and the index of that value. Else if the value doesn't exist, it returns an "Err" and indicates where we should insert the searched value instead. This is really convenient.

Thus we search for the new start time. (The new event to be inserted.)

In the case of "Ok", the index could either be a start time (even) or an end time (odd). If it's even, this indicates the event is conflicting with the new event. (Both have the same start time.) If it's odd, we know the new start time is equal to an end time.

```
end time (index) <= start time < start time(index+1)
```

This is okay according to the rules/constraints. Then we check if the new end time is less than or equal to the next start time (index+1) or if we're at the end of the vector. If either is true, we insert the times.

In the case of "Err", no value is equal to our new start time. If the index is even, this indicates our new start time is after an end time.

```
end time (index-1) < new start time < start time (index)
```

From there we check if we're at the end of the vector or if the new end time is less than or equal to the next start time (index). If so, we insert the new times and return true.

If the index is odd, this indicates the new start time is between a start time and an end time. Thus we do not insert and return false.

```
start time (index-1) < new start time < end time (index)
```

This was not my first approach... I tried to solve the problem using a BTreeMap... But it gets rather complicated. I really doubt you can solve this problem using a HashMap...

There's also the classic method of using a "bitmap" to indicate whether a time is used or not. This would be similar to a calendar of times and marking which times are used up. The upside is that it's (potentially) really fast. The downside is that it uses up a lot of space. (It's slow if you have to mark out a lot of time if the time span is really large.)

```Rust
use std::collections::VecDeque;

struct MyCalendar {
    events: VecDeque<i32>
}


/** 
 * `&self` means the method takes an immutable reference.
 * If you need a mutable reference, change it to `&mut self` instead.
 */
impl MyCalendar {

    fn new() -> Self {
        Self {
            events: VecDeque::new()
        }
    }
    
    fn book(&mut self, start: i32, end: i32) -> bool {
        match self.events.binary_search(&start) {
            Ok(index) => {
                //println!("ok{}",index);
                if index&1==0 {
                    return false;
                } else {
                    if index+1>=self.events.len() || end<=self.events[index+1] {
                        self.events.insert(index+1,end);
                        self.events.insert(index+1,start);
                        //println!("{:?}",self.events);
                        return true;
                    } else {
                        return false;
                    }
                }
            }
            Err(index) => {
                //println!("err{}",index);
                if index&1==0 {
                    if index>=self.events.len() || end<=self.events[index] {
                        self.events.insert(index,end);
                        self.events.insert(index,start);
                        //println!("{:?}",self.events);
                        return true;
                    } else {
                        return false;
                    }
                } else {
                    return false;
                }
            }
        }
        false
    }
}

/**
 * Your MyCalendar object will be instantiated and called as such:
 * let obj = MyCalendar::new();
 * let ret_1: bool = obj.book(start, end);
 */
```

# 2024-09-25
[2416. Sum of Prefix Scores of Strings](https://leetcode.com/problems/sum-of-prefix-scores-of-strings/)

"Acceptable"... The C solution required a whopping 700MB of memory and took 500ms to complete. The Rust solution was TLE.

With the constraints, minimum required memory is 1000 strings with 1000 chars. So that's only 1MB. Presumably the memory usage is cumulative? Hard to say...

Anyways, this is a naive implementation of a trie. (As recommended by the hints.)

The Rust version was TLE... Implementing a trie would be difficult in Rust... (Like every other tree structure.)

Technically speaking, hashes are memory efficient... Though it seems to be slow.

Using a BTreeMap for Rust seems to be barely good enough to pass at 1800ms with 600MB of memory.

```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

struct node {
    struct node * next[26];
    int val;
};

void insert(struct node *n, char *word) {
    n->val += 1;
    if (*word==0) {
        return;
    }
    int i = *word-'a';
    if (n->next[i] == NULL) {
        struct node *m = calloc(1,sizeof(struct node));
        n->next[i] = m;
    }
    insert(n->next[i],word+1);
};

int find(struct node *n, char *word) {
    if (*word==0) {
        return n->val;
    }
    int i = *word-'a';
    return n->val + find(n->next[i],word+1);
}

int* sumPrefixScores(char** words, int wordsSize, int* returnSize) {
    struct node *trie = calloc(1,sizeof(struct node));
    for (int i=0; i<wordsSize; i++) {
        insert(trie,words[i]);
    }
    int *out = malloc(wordsSize*sizeof(int));
    for (int i=0; i<wordsSize; i++) {
        out[i] = find(trie,words[i]);
        out[i] -= trie->val;
    }
    *returnSize = wordsSize;
    return out;
}
```

```Rust
use std::collections::HashMap;

impl Solution {
    pub fn sum_prefix_scores(words: Vec<String>) -> Vec<i32> {
        let mut map = HashMap::new();

        for w in words.iter() {
            for i in 1..=w.len() {
                map.entry(w[0..i].to_string())
                    .and_modify(|x| *x+=1)
                    .or_insert(1);
            }
        }

        let mut map2: HashMap<String, i32> = HashMap::new();
        let mut out = Vec::new();
        for w in words.iter() {
            match map2.get(w) {
                None => {
                    let mut v = 0;
                    for i in 1..=w.len() {
                        v += map.get(&w[0..i].to_string()).unwrap();
                    }
                    out.push(v);
                    map2.insert(w.to_string(),v);
                }
                Some(v) => {
                    out.push(*v);
                }
            }
        }

        out
    }
}
```

# 2024-09-24
[3043. Find the Length of the Longest Common Prefix](https://leetcode.com/problems/find-the-length-of-the-longest-common-prefix/)

Slow, barely passes.

```Rust
use std::cmp::min;

impl Solution {
    pub fn longest_common_prefix(arr1: Vec<i32>, arr2: Vec<i32>) -> i32 {
        let mut arr1 = arr1;
        let mut arr2 = arr2;
        arr1.sort(); arr1.dedup(); arr1.reverse();
        arr2.sort(); arr2.dedup(); arr2.reverse();
        let mut s1:Vec<String> = arr1.iter().map(|x| x.to_string()).collect();
        let mut s2:Vec<String> = arr2.iter().map(|x| x.to_string()).collect();

        let mut g = min(s1[0].len(),s2[0].len());
        while g>0 {
            let mut a1: Vec<_> = s1.iter().filter(|x| x.len()>=g).map(|x| (x[0..g].to_string(),1)).collect();
            let mut a2: Vec<_> = s2.iter().filter(|x| x.len()>=g).map(|x| (x[0..g].to_string(),2)).collect();
            a1.append(&mut a2);
            a1.sort_unstable();
            a1.dedup();
            for i in 0..a1.len()-1 {
                if a1[i].1 != a1[i+1].1 && a1[i].0 == a1[i+1].0 {
                    return g as i32;
                }
            }
            g -= 1;
        }
        0
    }
}
```

# 2024-09-23
[2707. Extra Characters in a String](https://leetcode.com/problems/extra-characters-in-a-string/)

Alright solution... Slow but doesn't use too much memory...

```Rust
impl Solution {
    pub fn min_extra_char(s: String, dictionary: Vec<String>) -> i32 {
        let mut xtra = vec![vec![s.len() as i32; s.len()]; s.len()];
        dfs(&s,&dictionary,&mut xtra,0);
        //println!("{:1?}",xtra);
        xtra[0][s.len()-1]
    }
}

fn dfs (s: &String, dict: &Vec<String>, xtra: &mut Vec<Vec<i32>>, i: usize) -> i32 {
    if i >= s.len() {
        return 0;
    }
    if xtra[i][s.len()-1] != s.len() as i32 {
        return xtra[i][s.len()-1];
    }
    let mut cost = Vec::new();
    for e in dict.iter() {
        if i+e.len()<=s.len() && *e==s[i..i+e.len()].to_string() {
            xtra[i][i+e.len()-1] = 0;
            cost.push(dfs(s,dict,xtra,i+e.len()));
        }
    }
    cost.push(1+dfs(s,dict,xtra,i+1));
    xtra[i][s.len()-1] = *cost.iter().min().unwrap();
    xtra[i][s.len()-1]
}
```

2024-09-22

TLE
```Rust
impl Solution {
    pub fn find_kth_number(n: i32, k: i32) -> i32 {
        let mut a = [1 as i64,10,100,1_000,10_000,100_000,1_000_000,
            10_000_000,100_000_000,1_000_000_000];
        let mut b = [1 as i64,10,100,1_000,10_000,100_000,1_000_000,
            10_000_000,100_000_000,1_000_000_000];
        let n = n as i64;
        let mut i = 1;
        let mut m = 0;
        while i<k {
            while a[m]<=n && i<k {
                //print!("{} ",a[m]);
                m += 1;
                i += 1;
            }
            if i==k && a[m]<=n {
                break;
            }
            m -= 1;
            for j in m..10 {
                a[j] += b[j-m];
            }
            while a[m]%10==0 || a[m]>n {
                m -= 1;
                a[m] += 1;
            }
        }
        //println!("{:?}",a);
        a[m] as i32
    }
}
```

```Rust
impl Solution {
    pub fn find_kth_number(n: i32, k: i32) -> i32 {
        let mut log = 1;
        let n = n as usize;
        let k = k as usize;
        let mut m = n;
        while m>9 {
            log += 1;
            m /= 10;
        }

        let mut a = vec![0; log];
        m = 1;
        for i in 0..log {
            a[i] = m;
            m *= 10;
        }
        let mut c = vec![0; log];

        let mut i = 1;
        m = 0;
        while i<k {
            while m<log-1 && i<k {
                print!("{} ",a[m]);
                i += 1;
                m += 1;
            }
            while a[m]<n && c[m]<10 && i<k {
                print!("{} ",a[m]);
                a[m] += 1;
                c[m] += 1;
                i += 1;
            }
            if i==k {
                break;
            }
            if a[m]==n {
                print!("{} ",a[m]);
                log -= 1;
                m -= 1;
                a[m] += 1;
                c[m] = 0;
                i += 1;
            } else if c[m]==10 {
                print!("{} ",a[m]);
                c[m] = 0;
                m -= 1;
                a[m] += 1;
                c[m] = 0;
                i += 1;
            }
        }

        a[m] as i32
    }
}
```

# 2024-09-21
[386. Lexicographical Numbers](https://leetcode.com/problems/lexicographical-numbers/)

Looks slow but gets 100% time.

```Rust
impl Solution {
    pub fn lexical_order(n: i32) -> Vec<i32> {
        let n = n as usize;
        let mut out = vec![0; n];

        let mut i = 0;
        let mut a = 1;
        while i<n {
            while a<=n {
                out[i] = a as i32;
                a *= 10;
                i += 1;
            }
            a /= 10;
            a += 1;
            while a%10 == 0 {
                a /= 10;
            }
        }

        out
    }
}
```

# 2024-09-20
[214. Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/)

Marked as hard but fairly straightforward. Strange... Others get TLE on this problem but this is pretty much a brute-force approach.

```Rust
impl Solution {
    pub fn shortest_palindrome(s: String) -> String {
        let t: String = s.chars().rev().collect();

        for i in 0..s.len() {
            let a = &t[i..s.len()];
            let b = &s[0..s.len()-i];
            if a==b {
                return t + &s[s.len()-i..s.len()];
            }
        }

        String::new()
    }
}
```

# 2024-09-19
[241. Different Ways to Add Parentheses](https://leetcode.com/problems/different-ways-to-add-parentheses/)

A bit of a doozy but the implementation is surprisingly "simple". The issue is trying to create unique permutations. On first inspection, it's permutations of which operator to perform first... Thus we can permute a sort of ordering... However, there are certain duplicate orderings...

This is related to https://en.wikipedia.org/wiki/Catalan_number#Applications_in_combinatorics . The article basically describes permutations of parentheses as well as the equivalent case of permutations of binary trees... The question of how to generate the permutation still remains...

Looking at https://en.wikipedia.org/wiki/Matrix_chain_multiplication , I eventually figured it out by splitting the expression into two parts recursively... From there each part has a certain amount of results then we simply iterate through both to generate new results.

This problem would classify as a dynamic programming (DP) problem. We deal with substrings of the expression and store the possible results from that substring. Thus we end up with a 2D array of an array of results. (A 3D array...)

Implementing this in C would be a nightmare... Managing memory would be insane.

DP is not necessary. We do get a tradeoff between time and memory. The DP version times at 0ms consistently. The non-DP version can sometimes get 100% time (0ms) and sometimes 100% (best) memory usage.

```Rust
impl Solution {
    pub fn diff_ways_to_compute(expression: String) -> Vec<i32> {
        let nums: Vec<i32> = expression
            .split(['+','-','*'])
            .map(|x| x.parse::<i32>().unwrap())
            .collect();
        let ops: Vec<char> = expression
            .matches(['+','-','*'])
            .map(|x| x.chars().nth(0).unwrap())
            .collect();
        
        //println!("{:?}",nums);
        //println!("{:?}",ops);

        let mut sub = vec![vec![Vec::new(); nums.len()]; nums.len()];

        dfs(&nums,&ops,&mut sub, 0,nums.len())
    }
}

fn dfs (nums: &Vec<i32>, ops: &Vec<char>, sub: &mut Vec<Vec<Vec<i32>>>, index: usize, len: usize) -> Vec<i32> {
    let mut v = sub[index][index+len-1].clone();
    if v.len()>0 {
        return v;
    }
    if len==1 {
        v.push(nums[index]);
        sub[index][index+len-1] = v.clone();
        return v;
    }
    for i in 1..len {
        let l = dfs(nums,ops,sub,index,i);
        let r = dfs(nums,ops,sub,index+i,len-i);
        match ops[index+i-1] {
            '+' => {
                for e in l.iter() {
                    for f in r.iter() {
                        v.push(*e + *f);
                    }
                }
            }
            '-' => {
                for e in l.iter() {
                    for f in r.iter() {
                        v.push(*e - *f);
                    }
                }
            }
            '*' => {
                for e in l.iter() {
                    for f in r.iter() {
                        v.push(*e * *f);
                    }
                }
            }
            _ => ()
        }
    }
	sub[index][index+len-1] = v.clone();
    return v;
}
```

Non-DP version:
```Rust
impl Solution {
    pub fn diff_ways_to_compute(expression: String) -> Vec<i32> {
        let nums: Vec<i32> = expression
            .split(['+','-','*'])
            .map(|x| x.parse::<i32>().unwrap())
            .collect();
        let ops: Vec<char> = expression
            .matches(['+','-','*'])
            .map(|x| x.chars().nth(0).unwrap())
            .collect();
        dfs(&nums,&ops,0,nums.len())
    }
}

fn dfs (nums: &Vec<i32>, ops: &Vec<char>, index: usize, len: usize) -> Vec<i32> {
    let mut v = Vec::new();
    if len==1 {
        v.push(nums[index]);
        return v;
    }
    for i in 1..len {
        let l = dfs(nums,ops,index,i);
        let r = dfs(nums,ops,index+i,len-i);
        match ops[index+i-1] {
            '+' => {
                for e in l.iter() {
                    for f in r.iter() {
                        v.push(*e + *f);
                    }
                }
            }
            '-' => {
                for e in l.iter() {
                    for f in r.iter() {
                        v.push(*e - *f);
                    }
                }
            }
            '*' => {
                for e in l.iter() {
                    for f in r.iter() {
                        v.push(*e * *f);
                    }
                }
            }
            _ => ()
        }
    }
    return v;
}
```

# 2024-09-18
[179. Largest Number](https://leetcode.com/problems/largest-number/)

Found a heuristic to get a solution.

<details>
<summary>Heuristic Spoiler</summary>
The heuristic is concatenating the two strings in both ways then simply comparing them. The alternative was to do a sort of search which can be quite painful to implement...
</details>

My first heuristic was simply to compare the two strings... And the other was duplicating the last character for comparison... It seems this problem can be solved greedily with the right heuristic.

A double input of {0,0} is such a weird edge case.

## Solution
```Rust
use std::cmp::Ordering;

impl Solution {
    pub fn largest_number(nums: Vec<i32>) -> String {
        let mut s: Vec<_> = nums.iter().map(|x| x.to_string()).collect();
        s.sort_unstable_by(|a,b| compare(b,a));
        if s[0] == "0" {
            return "0".to_string();
        }
        s.concat()
    }
}

fn compare(a: &String, b: &String) -> Ordering {
    if (a.len() == b.len()) {
        return a.cmp(b);
    }
    let c = a.clone() + b;
    let d = b.clone() + a;
    c.cmp(&d)
}
```

## Attempt 1
```Rust
use std::cmp::Ordering;
use std::cmp::Ordering::Less;
use std::cmp::Ordering::Equal;
use std::cmp::Ordering::Greater;
use std::cmp::min;

impl Solution {
    pub fn largest_number(nums: Vec<i32>) -> String {
        let mut s: Vec<_> = nums.iter().map(|x| x.to_string()).collect();
        s.sort_unstable_by(|a,b| compare(b,a));
        println!("{:?}",s);
        s.concat()
    }
}

fn compare(a: &String, b: &String) -> Ordering {
    let len = min(a.len(),b.len());
    let a = a.as_bytes();
    let b = b.as_bytes();
    for i in 0..len {
        match a[i].cmp(&b[i]) {
            Less => return Less,
            Equal => continue,
            Greater => return Greater
        }
    }
    if (a.len() == b.len()) {
        return Equal;
    } else if (a.len() < b.len()) {
        for i in len..b.len() {
            match a[len-1].cmp(&b[i]) {
                Less => return Less,
                Equal => continue,
                Greater => return Greater
            }
        }
    } else {
        for i in len..a.len() {
            match a[i].cmp(&b[len-1]) {
                Less => return Less,
                Equal => continue,
                Greater => return Greater
            }
        }
    }
    Equal
}
```

# 2024-09-17
[884. Uncommon Words from Two Sentences](https://leetcode.com/problems/uncommon-words-from-two-sentences/)

Just counting the amount of words and keeping words that have a count of one. (A count of one implies the word only appears in one sentence.)

```Rust
use std::collections::HashMap;

impl Solution {
    pub fn uncommon_from_sentences(s1: String, s2: String) -> Vec<String> {
        let mut words = HashMap::new();
        s1.split(' ').chain(s2.split(' ')).for_each(|x| {
            words.entry(x)
                .and_modify(|y| *y+=1)
                .or_insert(1);
            }
        );
        words.iter().filter(|(&k,&v)| v==1).map(|x| x.0.to_string()).collect()
    }
}
```

```Rust
use std::collections::HashMap;

impl Solution {
    pub fn uncommon_from_sentences(s1: String, s2: String) -> Vec<String> {
        let mut words = HashMap::new();
        s1.split(' ').for_each(|x| {
            words.entry(x)
                .and_modify(|y| *y+=1)
                .or_insert(1);
            }
        );
        s2.split(' ').for_each(|x| {
            words.entry(x)
                .and_modify(|y| *y+=1)
                .or_insert(1);
            }
        );
        //println!("{:?}",words);
        words.iter().filter(|(&k,&v)| v==1).map(|x| x.0.to_string()).collect()
    }
}
```

# 2024-09-16
[539. Minimum Time Difference](https://leetcode.com/problems/minimum-time-difference/)

Got a 100% speed and 100% memory run.

```C
int compare (int16_t *a, int16_t *b) {
    return *a - *b;
}

int findMinDifference(char** timePoints, int timePointsSize) {
    int n = timePointsSize;
    int16_t *a = malloc(n*sizeof(int16_t));
    for (int i=0; i<n; i++) {
        a[i] =
            (timePoints[i][0]-'0')*10*60 +
            (timePoints[i][1]-'0')*60 +
            (timePoints[i][3]-'0')*10 +
            (timePoints[i][4]-'0');
    }
    qsort(a,n,sizeof(int16_t),compare);
    int b;
    int min = 1440;
    for (int i=0; i<n-1; i++) {
        b = a[i+1] - a[i];
        if (b<min) {
            min = b;
        }
    }
    b = a[0]+1440-a[n-1];
    if (b<min) {
        min = b;
    }
    return min;
}
```

# 2024-09-15
[1371. Find the Longest Substring Containing Vowels in Even Counts](https://leetcode.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/)

Fast but uses a lot of memory.

```C
int findTheLongestSubstring(char* s) {
    int n = strlen(s)+1;
    uint8_t *a = calloc(n,sizeof(uint8_t));
    char *b = s;
    uint8_t *c = a;
    while (*b) {
        uint8_t d = 0;
        switch (*b) {
            case 'a': d = 1; break;
            case 'e': d = 2; break;
            case 'i': d = 4; break;
            case 'o': d = 8; break;
            case 'u': d = 16; break;
        }
        *(c+1) = *c ^ d;
        b++;
        c++;
    }
    /*
    for (int i=0; i<n; i++) {
        printf("%d ",a[i]);
    }
    */
    for (int i=n-1; i>0; i--) {
        for (int j=0; j<n-i; j++) {
            if (a[i+j] == a[j]) {
                return i;
            }
        }
    }
    return 0;
}
```

# 2024-09-14
[2419. Longest Subarray With Maximum Bitwise AND](https://leetcode.com/problems/longest-subarray-with-maximum-bitwise-and/)

Just finding the max and its longest sequence.

```C
int longestSubarray(int* nums, int numsSize) {
    int max = 0;
    for (int i=0; i<numsSize; i++) {
        if (max < nums[i]) {
            max = nums[i];
        }
    }
    int count = 0;
    int max_count = 0;
    for (int i=0; i<numsSize; i++) {
        if (nums[i]==max) {
            count++;
            if (max_count < count) {
                max_count = count;
            }
        } else {
            count = 0;
        }
    }

    return max_count;
}
```

# 2024-09-13
[1310. XOR Queries of a Subarray](https://leetcode.com/problems/xor-queries-of-a-subarray/)

```Rust
impl Solution {
    pub fn xor_queries(arr: Vec<i32>, queries: Vec<Vec<i32>>) -> Vec<i32> {
        let mut cumsum = vec![0; arr.len()+1];
        for (i,e) in arr.iter().enumerate() {
            cumsum[i+1] = cumsum[i] ^ arr[i];
        }
        queries.iter()
	        .map(|x| cumsum[x[0] as usize] ^ cumsum[x[1] as usize+1])
	        .collect()
    }
}
```

# 2024-09-12
[1684. Count the Number of Consistent Strings](https://leetcode.com/problems/count-the-number-of-consistent-strings/)

```C
int countConsistentStrings(char * allowed, char ** words, int wordsSize){
    int a = 0;
    char *c = allowed;
    while (*c) {
        a |= (1<<(*c-'a'));
        c++;
    }
    a = ~a;
    int count = 0;
    for (int i=0; i<wordsSize; i++) {
        c = words[i];
        int b = 0;
        while (*c) {
            b |= (1<<(*c-'a'));
            c++;
        }
        if ((a&b) == 0) {
            count++;
        }
    }

    return count;
}
```

# 2024-09-11
[2220. Minimum Bit Flips to Convert Number](https://leetcode.com/problems/minimum-bit-flips-to-convert-number/)

One-liners in both C and Rust.

## C
```C
int minBitFlips(int start, int goal) {
    /*
    uint32_t a = start ^ goal;
    int count;
    for (count=0; a; count++) {
        a &= a-1;
    }
    return count;
    */
    return __builtin_popcount(start ^ goal);
}
```

## Rust
```Rust
impl Solution {
    pub fn min_bit_flips(start: i32, goal: i32) -> i32 {
        i32::count_ones(start ^ goal) as i32
    }
}
```

# 2024-09-10
[2807. Insert Greatest Common Divisors in Linked List](https://leetcode.com/problems/insert-greatest-common-divisors-in-linked-list/)

```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */

int gcd(int a, int b) {
    if (a==0) {
        return b;
    } else if (b==0) {
        return a;
    }

    int a_k = 0;
    while (a&1 == 0) {
        a >>= 1;
        a_k++;
    }
    int b_k = 0;
    while (b&1 == 0) {
        b >>= 1;
        b_k++;
    }
    int k = (a_k<b_k) ? a_k : b_k;

    while (1) {
        if (a>b) {
            int t = a;
            a = b;
            b = t;
        }

        b -= a;

        if (b==0) {
            return a<<k;
        }

        while (b&1 == 0) {
            b >>= 1;
        }
    }
}

struct ListNode* insertGreatestCommonDivisors(struct ListNode* head){
    int len = 0;
    struct ListNode *curr;
    curr = head;

    while (curr!=NULL) {
        len++;
        curr = curr->next;
    }

    if (len<2) {
        return head;
    }

    struct ListNode *arr;
    arr = malloc((len-1)*sizeof(struct ListNode));

    struct ListNode *prev;
    prev = head;
    curr = head->next;
    int c = 0;

    while (curr != NULL) {
        int b = gcd(prev->val,curr->val);
        arr[c].val = b;
        arr[c].next = curr;
        prev->next = &arr[c];
        c++;
        prev = curr;
        curr = curr->next;
    }

    return head;
}
```

# 2024-09-09
[2326. Spiral Matrix IV](https://leetcode.com/problems/spiral-matrix-iv/)

Just a basic state machine. We're dealing with a linked list so it's hard to get optimizations. So we just simulate it.

A major optimization is reducing the amount of malloc calls. We can opt to make a single call to allocate the whole array rather than multiple calls for each row. This results in a speedup of at least 10ms with the total time taken 950ms. (Comparing the fastest run times.)

```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
/**
 * Return an array of arrays of size *returnSize.
 * The sizes of the arrays are returned as *returnColumnSizes array.
 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
 */
int** spiralMatrix(int m, int n, struct ListNode* head, int* returnSize, int** returnColumnSizes) {
    int **arr;
    arr = malloc(m*sizeof(int *));
    int *t;
    t = malloc(m*n*sizeof(int));
    for (int i=0; i<m; i++) {
        //arr[i] = malloc(n*sizeof(int));
        arr[i] = t;
        t += n;
        for (int j=0; j<n; j++){
            arr[i][j] = -1;
        }
    }
    int *cs;
    cs = malloc(m*sizeof(int));
    for (int i=0; i<m; i++) {
        cs[i] = n;
    }

    *returnSize = m;
    *returnColumnSizes = cs;

    int w = n;
    int h = m-1;
    int dir = 0;
    int x = 0;
    int y = 0;
    int c = 0;

    struct ListNode *curr;
    curr = head;

    while (curr!=NULL) {
        //printf("%d %d, ", y, x);
        if (dir==0) {
            arr[y][x] = curr->val;
            x++;
            c++;
            if (c>=w) {
                dir = 1;
                c = 0;
                w--;
                y++;
                x--;
            }
        } else if (dir==1) {
            arr[y][x] = curr->val;
            y++;
            c++;
            if (c>=h) {
                dir = 2;
                c = 0;
                h--;
                x--;
                y--;
            }
        } else if (dir==2) {
            arr[y][x] = curr->val;
            x--;
            c++;
            if (c>=w) {
                dir = 3;
                c = 0;
                w--;
                y--;
                x++;
            }
        } else if (dir==3) {
            arr[y][x] = curr->val;
            y--;
            c++;
            if (c>=h) {
                dir = 0;
                c = 0;
                h--;
                x++;
                y++;
            }
        }
        curr = curr->next;
    }

    return arr;
}
```

# 2024-09-08
[725. Split Linked List in Parts](https://leetcode.com/problems/split-linked-list-in-parts/)

```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
struct ListNode** splitListToParts(struct ListNode* head, int k, int* returnSize) {
    struct ListNode *curr;
    curr = head;
    int len = 0;
    while (curr!=NULL) {
        len++;
        curr = curr->next;
    }

    struct ListNode **arr;
    arr = calloc(k,sizeof(struct ListNode*));
    int a = len/k+1;
    int b = len%k;
    struct ListNode *prev;
    curr = head;
    prev = NULL;
    int c = 0;
    for (int i=0; i<b; i++) {
        arr[c] = curr;
        for (int j=0; j<a; j++) {
            prev = curr;
            curr = curr->next;
        }
        c++;
        prev->next = NULL;
    }
    a = len/k;
    b = k - (len%k);
    for (int i=0; i<b; i++) {
        if (a<1) {
            arr[c] = NULL;
        } else {
            arr[c] = curr;
            for (int j=0; j<a; j++) {
                prev = curr;
                curr = curr->next;
            }
            prev->next = NULL;
        }
        c++;
    }

    *returnSize = k;
    return arr;
}
```

# 2024-09-07
[1367. Linked List in Binary Tree](https://leetcode.com/problems/linked-list-in-binary-tree/)

Tried to compare it ad-hoc... The logic becomes complicated. (We only want to visit every node only once.) The second version would be the "naive" method but uses a lot of memory.

The way LeetCode measures the speed is all over the place... Got one run out of 6 beating 100% of submissions.

Presumably the fastest way is to have a buffer. To reduce memory usage, the solution should be iterative instead of recursive.
## Works

```C
bool dfs(uint8_t *pat, int pn, struct TreeNode* node, uint8_t *buf, int bn) {
    if(bn>=pn) {
        int i = 0;
        int j = bn-pn;
        while (i<pn) {
            if (pat[i] != buf[j]) {
                break;
            }
            i++;
            j++;
        }
        if (i==pn) {
            return true;
        }
    }

    if (node == NULL) {
        return false;
    }
    buf[bn] = node->val;
    if (dfs(pat,pn,node->left,buf,bn+1)) {
        return true;
    }
    if (dfs(pat,pn,node->right,buf,bn+1)) {
        return true;
    }

    return false;
}

bool isSubPath(struct ListNode* head, struct TreeNode* root) {
    uint8_t buf[2500];
    uint8_t pat[100];

    int pn = 0;
    while (head != NULL) {
        pat[pn] = head->val;
        pn++;
        head = head->next;
    }

    return dfs(pat,pn,root,buf,0);
}
```

## Fail

```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
bool dfs(struct ListNode *head, struct TreeNode* root, struct ListNode *curr) {
    if (curr == NULL) {
        return true;
    }
    if (root == NULL) {
        return false;
    }
    printf("%d ",curr->val);

    if (root->val != curr->val) {
        /*
        if (head != curr) {
            if (dfs(head,root,head)) {
                return true;
            }
            return false;
        }
        */
        if (head == curr) {
            if (dfs(head,root->left,head)) {
                return true;
            }
            if (dfs(head,root->right,head)) {
                return true;
            }
            return false;
        } else {
            if (dfs(head,root,head)) {
                return true;
            }
            return false;
        }
    } else {
        if (dfs(head,root->left,curr->next)) {
            return true;
        }
        if (dfs(head,root->right,curr->next)) {
            return true;
        }
    }
    return false;
}

bool isSubPath(struct ListNode* head, struct TreeNode* root) {
    return dfs(head,root,head);
}

```

# 2024-09-06
[3217. Delete Nodes From Linked List Present in Array](https://leetcode.com/problems/delete-nodes-from-linked-list-present-in-array/)

```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* modifiedList(int* nums, int numsSize, struct ListNode* head) {
    bool *b = calloc(100001,1);

    for (int i=0; i<numsSize; i++) {
        b[nums[i]] = true;
    }

    struct ListNode l;
    l.val = 0;
    l.next = head;

    struct ListNode *curr, *last;
    last = &l;
    curr = &l;

    while (curr) {
        if (!b[curr->val]) {
            last = curr;
        } else {
            last->next = curr->next;
        }
        curr = curr->next;
    }

    return l.next;
}
```
# 2024-09-05
[2028. Find Missing Observations](https://leetcode.com/problems/find-missing-observations/)

```Rust
impl Solution {
    pub fn missing_rolls(rolls: Vec<i32>, mean: i32, n: i32) -> Vec<i32> {
        let count = rolls.len() + n as usize;
        let total = count as i32 * mean;

        let left = total - rolls.iter().sum::<i32>();
        if left < n {
            return Vec::new();
        }
        if left > n*6 {
            return Vec::new();
        }
        let b = left/n;
        let r = (left%n) as usize;

        let mut out = vec![b;n as usize];
        for i in 0..r {
            out[i] += 1;
        }
        out
    }
}
```

# 2024-09-04
[874. Walking Robot Simulation](https://leetcode.com/problems/walking-robot-simulation/)

Doesn't look pretty but it works.

```Rust
use std::collections::HashMap;

impl Solution {
    pub fn robot_sim(commands: Vec<i32>, obstacles: Vec<Vec<i32>>) -> i32 {
        let mut xobs = HashMap::new();
        let mut yobs = HashMap::new();

        for e in obstacles.iter() {
            let x = e[0];
            let y = e[1];
            match xobs.get_mut(&x) {
                None => {
                    let mut a = Vec::new();
                    a.push(y);
                    xobs.insert(x,a);
                }
                Some(a) => {
                    a.push(y);
                    a.sort();
                }
            }
            match yobs.get_mut(&y) {
                None => {
                    let mut a = Vec::new();
                    a.push(x);
                    yobs.insert(y,a);
                }
                Some(a) => {
                    a.push(x);
                    a.sort();
                }
            }
        }

        let mut dir = 0;
        let mut x = 0;
        let mut y = 0;
        let mut max = 0;

        for c in commands.iter() {
            match c {
                -2 => {
                    dir = (dir+3)%4;
                }
                -1 => {
                    dir = (dir+1)%4;
                }
                1..=9 => {
                    match dir {
                        0 => {
                            //print!("n ");
                            match xobs.get(&x) {
                                None => {
                                    y += c;
                                }
                                Some(a) => {
                                    let mut i = 0;
                                    let mut yn = y+c;
                                    while i<a.len() && y>=a[i] {
                                        i += 1;
                                    }
                                    if i<a.len() && yn>a[i]-1 {
                                        yn = a[i]-1;
                                    }
                                    y = yn;
                                }
                            }
                        }
                        1 => {
                            //print!("e ");
                            match yobs.get(&y) {
                                None => {
                                    x += c;
                                }
                                Some(a) => {
                                    let mut i = 0;
                                    let mut xn = x+c;
                                    while i<a.len() && x>=a[i] {
                                        i += 1;
                                    }
                                    if i<a.len() && xn>a[i]-1 {
                                        xn = a[i]-1;
                                    }
                                    x = xn;
                                }
                            }
                        }
                        2 => {
                            //print!("s ");
                            match xobs.get(&x) {
                                None => {
                                    y -= c;
                                }
                                Some(a) => {
                                    let mut i = a.len()-1;
                                    let mut yn = y-c;
                                    while i<a.len() && y<=a[i] {
                                        i -= 1;
                                    }
                                    if i<a.len() && yn<a[i]+1 {
                                        yn = a[i]+1;
                                    }
                                    y = yn;
                                }
                            }
                        }
                        3 => {
                            //print!("w ");
                            match yobs.get(&y) {
                                None => {
                                    x -= c;
                                }
                                Some(a) => {
                                    let mut i = a.len()-1;
                                    let mut xn = x-c;
                                    while i<a.len() && x<=a[i] {
                                        i -= 1;
                                    }
                                    if i<a.len() && xn<a[i]+1 {
                                        xn = a[i]+1;
                                    }
                                    x = xn;
                                }
                            }
                        }
                        _ => ()
                    }
                    //print!("{},{} ",x,y);
                    let dis = x*x+y*y;
                    if dis>max {
                        max = dis;
                    }
                }
                _ => ()
            }
        }

        /*
        println!("{},{}",x,y);
        println!("x {:?}",xobs);
        println!("y {:?}",yobs);
        */

        max
    }
}
```
# 2024-09-03
[1945. Sum of Digits of String After Convert](https://leetcode.com/problems/sum-of-digits-of-string-after-convert/)

```Rust
impl Solution {
    pub fn get_lucky(s: String, k: i32) -> i32 {
        let a:Vec<String> = s.chars()
            .map(|x| format!("{}",x as u8-b'a'+1))
            .collect();
        let mut b = a.concat();
        let mut c = 0;

        for i in 0..k {
            c = digit_sum(b);
            b = format!("{}",c);
        }

        c
    }
}

fn digit_sum(s: String) -> i32 {
    s.bytes().map(|x| (x-b'0') as i32).sum()
}
```

# 2024-09-02
[1894. Find the Student that Will Replace the Chalk](https://leetcode.com/problems/find-the-student-that-will-replace-the-chalk/)

```Rust
impl Solution {
    pub fn chalk_replacer(chalk: Vec<i32>, k: i32) -> i32 {
        let total:i64 = chalk.iter().map(|x| *x as i64).sum();
        let mut k = if (k as i64)<total {
            k
        } else {
            k%(total as i32)
        };
        let mut i = 0;
        while chalk[i]<=k {
            k -= chalk[i];
            i += 1;
        }

        i as i32
    }
}
```

# 2024-09-01
[2022. Convert 1D Array Into 2D Array](https://leetcode.com/problems/convert-1d-array-into-2d-array/)

```Rust
impl Solution {
    pub fn construct2_d_array(original: Vec<i32>, m: i32, n: i32) -> Vec<Vec<i32>> {
        if (n*m) as usize != original.len() {
            return Vec::new();
        }
        original[..].chunks_exact(n as usize).map(|x| x.try_into().unwrap()).collect()
    }
}
```

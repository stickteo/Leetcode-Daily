# 2024-10-18
[2044. Count Number of Maximum Bitwise-OR Subsets](https://leetcode.com/problems/count-number-of-maximum-bitwise-or-subsets/)

A fast solution but uses a lot of memory. We basically calculate the value for every subset and then count the subsets which have the maximum value.

To calculate the max, we simply calculate the cumulative OR of every number.

To calculate each subset, we use our previous calculations to calculate our next subset value. In essence, we iterate through 0 to 2^nums.len(). (The total amount of subsets is 2 to the power of the amount of numbers we have; Each number is either included in the subset or not included.) Thus each index in binary represents whether each number is included in the subset or not.

Using the following example input: (3,1,2)
```
           3 1 2
0 = 000 =>       => 0
1 = 001 =>     2 => 2
2 = 010 =>   1   => 1
3 = 011 =>   1|2 => 3
4 = 100 => 3     => 3
5 = 101 => 3|  2 => 3
6 = 110 => 3|1   => 3
7 = 111 => 3|1|2 => 3
```

We can see that for indexes 4 to 7, we simply OR 3 with the results from 0 to 3.

In some sense this can be called "dynamic programming" where we use the previous calculations to calculate new values.

## Bruteforce
```Rust
impl Solution {
    pub fn count_max_or_subsets(nums: Vec<i32>) -> i32 {
        let mut subs = vec![0; 1<<nums.len()];
        for i in 0..nums.len() {
            subs[1<<i] = nums[i];
            for j in 0..(1<<i) {
                subs[(1<<i)+j] = nums[i]|subs[j];
            }
        }
        //println!("{:?}",subs);
        let mut max = 0;
        nums.iter().for_each(|x| max|=x);
        subs.iter().filter(|x| **x==max).count() as i32
    }
}
```

# 2024-10-17
[670. Maximum Swap](https://leetcode.com/problems/maximum-swap/)

Starting at each digit position, we want to make sure we have the best possible value. A good test case would be "9937". Just checking the first two digits would not be enough.

For each position, we want to get the max of the remaining digits. If the digit is the max, we go to the next position. Else, we find the max of the remaining digits and swap the max with our current position.

Since we're calculating the max at each position, we might as well sort.

For the code, we first create a string from the number. Duplicate that string and sort it in descending order. Then we compare each digit in both strings. If we reach the end, we just return our input number. Else, we find the same digit by starting from the end of our unsorted string. Finally we perform the swap and covert the string back into a number.

```Rust
impl Solution {
    pub fn maximum_swap(num: i32) -> i32 {
        let mut s = num.to_string().into_bytes();
        let mut t = s.clone();
        t.sort_unstable_by(|a,b| b.cmp(a));
        let mut i = 0;
        while i<s.len() && s[i]==t[i] {
            i += 1;
        }
        if i==s.len() {
            return num;
        }
        let mut j = s.len()-1;
        while t[i]!=s[j] {
            j -= 1;
        }
        s[j] = s[i];
        s[i] = t[i];
        String::from_utf8(s).unwrap().parse().unwrap()
    }
}
```

# 2024-10-16
[1405. Longest Happy String](https://leetcode.com/problems/longest-happy-string/)

The code looks long but it's pattern matching and a lot of repetition.

For this problem, we're tasked with generating a "longest happy string".

Let's go through this solution first. We first use the character with the largest amount. For example with a=1, b=1, c=7, we have the most amount of 'c's. We use this first by pushing it into our initial vector. Then we set it as the "previous" character used.

Then we iterate matching against the previous character. Then we compare which of the remaining amounts are larger... Using the larger one, and decrementing its count.

For this example, we get a vector with a pattern: "cacbc". The loop ends when we see the two remaining amounts are both 0.

From there, we iterate on our pattern to generate the final string. We simply add another character if we have remaining characters to use. Thus "cacbc" becomes "ccaccbcc" with one 'c' left unused. Suppose c=5, we would get "ccaccbc" instead.

## Solution
```Rust
impl Solution {
    pub fn longest_diverse_string(a: i32, b: i32, c: i32) -> String {
        let mut nums = vec![a,b,c];
        let mut prev = if a<b {
            if b<c {2} else {1}
        } else {
            if a<c {2} else {0}
        };

        let mut v = vec![prev];
        nums[prev] -= 1;

        loop { match prev {
            0 => if nums[1]<nums[2] {
                v.push(2); nums[2] -= 1; prev = 2;
            } else {
                if nums[1] == 0 { break; }
                v.push(1); nums[1] -= 1; prev = 1;
            }
            1 => if nums[0]<nums[2] {
                v.push(2); nums[2] -= 1; prev = 2;
            } else {
                if nums[0] == 0 { break; }
                v.push(0); nums[0] -= 1; prev = 0;
            }
            2 => if nums[0]<nums[1] {
                v.push(1); nums[1] -= 1; prev = 1;
            } else {
                if nums[0] == 0 { break; }
                v.push(0); nums[0] -= 1; prev = 0;
            }
            _ => {}
        }}
        //println!("{:?}",v);

        let mut s = String::new();
        for e in v.iter() {
            if nums[*e] > 0 {
                match e {
                    0 => s.push_str("aa"),
                    1 => s.push_str("bb"),
                    2 => s.push_str("cc"),
                    _ => {}
                }
                nums[*e]-=1;
            } else {
                match e {
                    0 => s.push('a'),
                    1 => s.push('b'),
                    2 => s.push('c'),
                    _ => {}
                }
            }
        }
        s
    }
}
```

## Draft
```Rust
impl Solution {
    pub fn longest_diverse_string(a: i32, b: i32, c: i32) -> String {
        let mut nums = vec![a,b,c];
        let mut prev = if a<b {
            if b<c {2} else {1}
        } else {
            if a<c {2} else {0}
        };

        let mut v = vec![(prev,1)];
        nums[prev] -= 1;

        loop {
            match prev {
                0 => if nums[1]<nums[2] {
                    v.push((2,1));
                    nums[2] -= 1;
                    prev = 2;
                } else {
                    if nums[1] == 0 {
                        break;
                    }
                    v.push((1,1));
                    nums[1] -= 1;
                    prev = 1;
                }
                1 => if nums[0]<nums[2] {
                    v.push((2,1));
                    nums[2] -= 1;
                    prev = 2;
                } else {
                    if nums[0] == 0 {
                        break;
                    }
                    v.push((0,1));
                    nums[0] -= 1;
                    prev = 0;
                }
                2 => if nums[0]<nums[1] {
                    v.push((1,1));
                    nums[1] -= 1;
                    prev = 1;
                } else {
                    if nums[0] == 0 {
                        break;
                    }
                    v.push((0,1));
                    nums[0] -= 1;
                    prev = 0;
                }
                _ => {}
            }
        }
        println!("{:?}",v);
        
        for e in v.iter_mut() {
            if nums[e.0] > 0 {
                e.1 += 1;
                nums[e.0]-=1;
            }
        }
        println!("{:?}",v);

        String::new()
    }
}
```

# 2024-10-15
[2938. Separate Black and White Balls](https://leetcode.com/problems/separate-black-and-white-balls/)

You can see how my solutions iteratively get better...

In Rust, when you collect(), you create a new vector and that significantly takes more time. (More memory needs to be allocated.)

Starting at the beginning, we first notice optimal swaps must be swapping 1s to the right with 0s. Swapping 1s and 1s will not be optimal (nothing changes).

Then we notice the rightmost '1' must be at the last index. The second rightmost '1' would be at the second last index... And so forth...

For example:
```
0 1 2 3 4
---------
1 0 1 0 0
1---->1
    1-->1
0 0 0 1 1
```

So for "10100" to change into "00011". The starting indexes of 1s are: (0,2) and the ending indexes are: (3,4). So our number of swaps are (3-0)+(4-2) = 5.

This forms the basis for our bruteforce algorithm... It's not very smart... (Trying to sort the 0s out from the 1s...)

The faster solution avoids including 0s. But it's still not very good. However, I was getting close by calculating the string's length minus the amount of 1s... (s.len()-a.len()).

So... The s.len() is the amount of 0s and 1s... Then we're subtracting the amount of 1s from it... So in the best solution, we simply calculate the amount of 0s. Naming it C, C represents the position of the leftmost '1' in the final string. Looking back at the example "10100", C = 3. From there we can easily generate the ending indexes by simply adding 1 to C for each iteration. Done. Easy.

## Best
```Rust
impl Solution {
    pub fn minimum_steps(s: String) -> i64 {
        let mut c = s.chars().filter(|x| *x=='0').count();
        let mut sum = 0 as i64;
        s.chars().enumerate().filter(|x| x.1=='1')
        .for_each(|x| {
            sum += (c-x.0) as i64;
            c += 1;
        });
        sum
    }
}
```

## Faster
```Rust
impl Solution {
    pub fn minimum_steps(s: String) -> i64 {
        let a: Vec<i32> = s.chars()
        .enumerate().filter(|x| x.1=='1')
        .map(|x| x.0 as i32)
        .collect();
        //println!("{:?}",a);
        let b = s.len()-a.len();
        a.iter().enumerate()
        .map(|x| (x.0+b-*x.1 as usize) as i64)
        .sum()
    }
}
```

## Slow Bruteforce
```Rust
impl Solution {
    pub fn minimum_steps(s: String) -> i64 {
        let mut a: Vec<i32> = s.chars()
        .enumerate().map(|x|
            if x.1 == '0' {
                -1
            } else {
                x.0 as i32
            }
        ).collect();
        a.sort_unstable();
        //println!("{:?}",a);
        a.iter().enumerate().map(|x|
            if *x.1 == -1 {
                0
            } else {
                x.0 as i64 - *x.1 as i64
            }
        ).sum()
    }
}
```

# 2024-10-14
[2530. Maximal Score After Applying K Operations](https://leetcode.com/problems/maximal-score-after-applying-k-operations/)

A fairly straight forward solution... Though maybe not the fastest. A simple implementation though.

We want a sorted array so we can always get the largest number to add to our sum. However, we also need to update that value and somehow keep our array sorted.

A simple way is to use a BinaryHeap where we can just pop the max value. Update that value. Then push that value back in. Finally, just iterate for k steps.

To get the ceil() function, we can just add one less than the divisor before dividing.

In terms of speed, it's slower than most of the submissions... The runtimes range from 30ms to 36ms for 6 runs, with an average of around 34ms. The best runtime is 29ms. So this solution would be around 10% slower than the fastest... 33ms beats 66.67%... There's seems to be 3 main bins Leetcode separates the solutions into...

Not much of a spoiler but looking at the hints basically tells you to use a heap.

```Rust
use std::collections::BinaryHeap;
impl Solution {
    pub fn max_kelements(nums: Vec<i32>, k: i32) -> i64 {
        let mut k = k;
        let mut heap = BinaryHeap::new();
        for e in nums {
            heap.push(e);
        }
        let mut sum = 0 as i64;
        while k>0 {
            let v = heap.pop().unwrap();
            sum += v as i64;
            heap.push((v+2)/3);
            k-=1;
        }
        sum
    }
}
```

# 2024-10-13
[632. Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/)

Since each list is already sorted, we always know the min and max of each list. (Otherwise we will have to sort it first.)

The main problem to solve is trying to construct the range. There was some brainstorming trying to figure it out... Mainly thinking about minimums and maximums and drawing out an algorithm on paper...

<details>
<summary>Spoiler!</summary>
The key insight is that we can construct the range from the maximums from each list.

We maintain the range so we can always calculate max-min. (The range is the same length as the amount of lists.) We attach the index so we can lookup which list to pop from next.

We update our best if the current range is smaller. Then, we pop the max value from our range; This will also give us the index of the list to get the next value from. Thus, we insert that value into our range. From there, we just repeat until one of our lists is empty.

There are a variety of ways to maintain our range... For this problem, we're going to use a BTreeSet. Inserts (insert()), min (first()), and max (last()) should all take at most O(logn).

In comparison, using a Vector means inserts will take O(n) time.

With the constraints, we have K lists (3500) with each list having at most 50 values. The worst case would be 3500 * 50 = 175,000 iterations. Using a Vector would be roughly on the order of 3500*3500*50 = 612,500,000 operations (mostly writes)... Whereas using a BTreeSet means log2(3500)*3500*50 = 12*3500*50 = 2,100,000 operations (inserts/writes)... This means a BTreeSet needs to be around 300 times slower per operation before a Vector would be advantageous...

I'm just not sure which data structure would be as fast... LinkedLists could be a simple alternative... Reads are inefficient for LinkedLists but inserts are much faster than Vectors...

P.S. The discussion does indicate a sort of sliding window with all elements sorted. It was one of my initial ideas... It could actually work... I just need to maintain a ~~Boolean array~~ count of whether my window has that list or not... That might be faster...
</details>

## "Typical" Solution
```Rust
use std::collections::BTreeSet;

impl Solution {
    pub fn smallest_range(nums: Vec<Vec<i32>>) -> Vec<i32> {
        // edge cases
        if nums.len() == 1 {
            let a = nums[0][0];
            return vec![a,a];
        }

        // initialize
        let mut nums = nums;
        let mut set = BTreeSet::new();
        for (i,e) in nums.iter_mut().enumerate() {
            match e.pop() {
                None => {
                    println!("error!");
                }
                Some(a) => {
                    set.insert((a,i));
                }
            }
        }
        //println!("{:?}",set);
        let mut best = (set.first().unwrap().0,set.last().unwrap().0);

        // main algorithm
        while let Some(a) = set.pop_last() {
            let (val,list) = a;
            let curr = (set.first().unwrap().0,val);
            if best.1-best.0 >= curr.1-curr.0 {
                best = curr;
            }
            match nums[list].pop() {
                None => break,
                Some(b) => {
                    set.insert((b,list));
                }
            }
        }
        vec![best.0,best.1]
    }
}
```

## Idea 1
```Rust
use std::collections::BinaryHeap;

impl Solution {
    pub fn smallest_range(nums: Vec<Vec<i32>>) -> Vec<i32> {
        let mut lens: Vec<usize> = nums.iter().map(|x| x.len()).collect();
        println!("{:?}",lens);
        let mut heap = BinaryHeap::new();
        for (list,e) in nums.iter().enumerate() {
            for val in e.iter() {
                heap.push((val,list));
            }
        }
        println!("{:?}",heap);
        let mut max = 0;
        while match heap.peek() {
            None => false,
            Some(&(val,list)) => {
                print!("{},{} ",val,list);
                if lens[list] > 1 {
                    lens[list] -= 1;
                    true
                } else {
                    max = *val;
                    false
                }
            }
        } {
            heap.pop();
        }
        println!("{:?}",lens);

        vec![1,max]
    }
}
```

# 2024-10-12
[2406. Divide Intervals Into Minimum Number of Groups](https://leetcode.com/problems/divide-intervals-into-minimum-number-of-groups/)

First we sort. Then we greedily merge an interval with the earliest possible end time. This is done by storing the end times for each group.

This is quite slow in comparison to the other results... Using a VecDeque at least halves the amount of time... This indicates inserting and removal takes a lot of time.

Looking at the hints is quite a weird experience... Somehow the minimum number of groups is equivalent to maximum number of overlapping intervals...

The Greedy Heap approach is eerily similar to the bruteforce approach though it's actually different. We basically pop out end times that are less than our new start time... Then we push our new end time. Finally we check if the length is the max or not. We iterate until we process all of the intervals. Using a heap means our run time is O(n logn). (We have O(logn) operations and iterate over n elements.)

In some sense the "puzzle pieces" are:
- sorting the intervals
- comparing the start times to stored end times
- reframing the problem as max overlapping intervals "at some point"

## Greedy Heap (Looked at hints)
```Rust
use std::collections::BinaryHeap;
impl Solution {
    pub fn min_groups(intervals: Vec<Vec<i32>>) -> i32 {
        let mut ints = intervals;
        ints.sort_unstable();
        let mut heap = BinaryHeap::new();
        let mut max = 0;
        for e in ints.iter() {
            while match heap.peek() {
                None => false,
                Some(a) => -e[0] < *a
            } {
                heap.pop();
            }
            heap.push(-e[1]);
            max = max.max(heap.len());
        }
        max as i32
    }
}
```

## Greedy Bruteforce (No comments)
```Rust
use std::collections::VecDeque;
impl Solution {
    pub fn min_groups(intervals: Vec<Vec<i32>>) -> i32 {
        let mut ints = intervals;
        ints.sort_unstable();
        let mut groups = VecDeque::new();
        for e in ints.iter() {
            let i = groups.partition_point(|x| x<&e[0]);
            if i!=0 {
                groups.remove(i-1);
            }
            let j = groups.partition_point(|x| x<&e[1]);
            groups.insert(j,e[1]);
        }
        //print!("{:?}",groups);
        groups.len() as i32
    }
}
```

## Greedy
```Rust
impl Solution {
    pub fn min_groups(intervals: Vec<Vec<i32>>) -> i32 {
        let mut ints = intervals;
        ints.sort_unstable();
        let mut groups = Vec::new();
        for e in ints.iter() {
            /*
            let i = match groups.binary_search(&e[0]) {
                Ok(a) => a,
                Err(a) => a
            };
            */
            let i = groups.partition_point(|x| x<&e[0]);
            /*
            if i==0 {
                let j = match groups.binary_search(&e[1]) {
                    Ok(a) => a+1,
                    Err(a) => a
                };
                groups.insert(j,e[1]);
            } else {
                groups.remove(i-1);
                let j = match groups.binary_search(&e[1]) {
                    Ok(a) => a+1,
                    Err(a) => a
                };
                groups.insert(j,e[1]);
            }
            */
            if i!=0 {
                groups.remove(i-1);
            }
            /*
            let j = match groups.binary_search(&e[1]) {
                Ok(a) => a,
                Err(a) => a
            };
            */
            let j = groups.partition_point(|x| x<&e[1]);
            groups.insert(j,e[1]);
        }
        //print!("{:?}",groups);
        groups.len() as i32
    }
}
```

## Greedy TLE
```C
int compare (int **a, int **b) {
    if (*a[0] == *b[0]) {
        return *a[1] - *b[1];
    } else {
        return *a[0] - *b[0];
    }
}

int minGroups(int** intervals, int intervalsSize, int* intervalsColSize) {
    qsort(intervals,intervalsSize,sizeof(int **),compare);

    /*
    for (int i=0; i<intervalsSize; i++) {
        printf("%d,%d ",intervals[i][0],intervals[i][1]);
    }
    printf("\n");
    */

    int groups=1;
    for (int i=1; i<intervalsSize; i++) {
        int j=0;
        while (j<groups) {
            if (intervals[i][0]>intervals[j][1]) {
                intervals[j][1] = intervals[i][1];
                break;
            }
            j++;
        }
        if (j>=groups) {
            intervals[groups][0] = intervals[i][0];
            intervals[groups][1] = intervals[i][1];
            groups++;
        }
    }
    
    /*
    for (int i=0; i<groups; i++) {
        printf("%d,%d ",intervals[i][0],intervals[i][1]);
    }
    */

    return groups;
}
```

# 2024-10-11
[1942. The Number of the Smallest Unoccupied Chair](https://leetcode.com/problems/the-number-of-the-smallest-unoccupied-chair/)

Slow and uses a lot of memory!

```Rust
use std::collections::BinaryHeap;
use std::collections::HashMap;

impl Solution {
    pub fn smallest_chair(times: Vec<Vec<i32>>, target_friend: i32) -> i32 {
        let mut heap = BinaryHeap::new();
        for (i,e) in times.iter().enumerate() {
	        if times[target_friend as usize][0] < e[0] {
                continue;
            }
            // negative times
            // - early times will be the max
            //   and taken out first in max heap
            // negative index to indicate arrival
            // - leaving is prioritized with a positive index
            heap.push((-e[0] as i32, 0-(i as i32+1)));
            heap.push((-e[1] as i32, i as i32+1));
        }

        let mut chairs = Vec::new();
        let mut map = HashMap::new();
        while !heap.is_empty() {
            let (t,f) = heap.pop().unwrap();
            //print!("{} ",f);
            if f<0 {
                // arrive
                let mut c = chairs.len();
                for (i,e) in chairs.iter().enumerate() {
                    if *e == 0 {
                        c = i;
                        break;
                    }
                }
                if f == target_friend+1 {
                    return c as i32;
                }
                if c == chairs.len() {
                    chairs.push(0-f);
                    map.insert(0-f,c);
                } else {
                    chairs[c] = 0-f;
                    map.insert(0-f,c);
                }
                //print!("{},{} ",f,c);
            } else {
                // leave
                chairs[map[&f]] = 0;
                //print!("{},{} ",f,map[&f]);
            }
        }

        map[&(target_friend+1)] as i32
    }
}
```

# 2024-10-10
[962. Maximum Width Ramp](https://leetcode.com/problems/maximum-width-ramp/)

Slow solution. My loop logic needs to be better. I'm strangely bad at these sort of questions.

Anyways, the first simplest solution got TLE. The worst test case are a series of descending numbers... (I.e. 9,8,7,6,5,4,3,2,1.)

This algorithm is greedy. There are two things we need to optimize: the width and minimum/maximum value.

We first create an array of minimums and maximums.

A minimum number is when it's the smallest number with the smallest index. Suppose at index N with number A, if there's an index M with number B such that M < N and B <= A. This means number B at index M will always give a better width.

Similarly, maximum numbers are the largest numbers with the largest index.

For example:
```
nums = [9,8,1,0,1,9,4,0,4,1]
mins = [9,8,1,0            ]
maxs = [          9,4,    1]
```

Once the index is over 3, the best minimum is 0. Similarly, once the index is under 5, the best maximum is 9.

After getting the max and min arrays, we "zig-zag" between the maximums and minimums to find the best possible width. "Zig-zagging" is possible since we have sorted values; If we know A < B and C < A, we don't have to check that C < B. (We have to establish an "ordering" or create that associative property that allows us to compare a few values.)

Perhaps a visualization may help, minimums are at the left while maximums are at top.
```
  1 4 9
9 > > =
8   > <
1 = <
0 <
```

Minimums need to be less than or equal to the maximums to be valid. With the zig-zag approach, we don't have to compare every number. Ideally, we want to minimize the amount of comparisons we have to do.

```C
int maxWidthRamp(int* nums, int numsSize) {
    int *min = malloc(numsSize*sizeof(int));
    int *max = malloc(numsSize*sizeof(int));

    int minl;
    int maxl;

    min[0] = 0;
    minl = 1;
    for (int i=1; i<numsSize; i++) {
        if (nums[i]<nums[min[minl-1]]) {
            min[minl] = i;
            minl++;
        }
    }

    max[0] = numsSize-1;
    maxl = 1;
    for (int i=numsSize-2; i>=0; i--) {
        if (nums[i]>nums[max[maxl-1]]) {
            max[maxl] = i;
            maxl++;
        }
    }

    /*
    for (int i=0; i<minl; i++) {
        printf("%d ",nums[min[i]]);
    }
    printf("\n");

    for (int i=0; i<maxl; i++) {
        printf("%d ",nums[max[i]]);
    }
    printf("\n");
    */

    /*
    for (int i=0; i<minl; i++) {
        for (int j=0; j<maxl; j++) {
            if (max[j]-min[i] <= w) {
                break;
            }
            if (nums[min[i]] <= nums[max[j]]) {
                w = max[j]-min[i];
                break;
            }
        }
    }
    */
    int j=0;
    while (nums[0]>nums[max[j]]) {
        j++;
    }
    int w=max[j];
    for (int i=1; i<minl; i++) {
        while (j>0 && nums[max[j]]>nums[min[i]]) {
            j--;
        }
        if (nums[max[j]]<nums[min[i]]) {
            j++;
        }
        //printf("%d %d\n",nums[min[i]],nums[max[j]]);
        if (max[j]-min[i]>w) {
            //printf("* %d %d\n",nums[min[i]],nums[max[j]]);
            w = max[j]-min[i];
        }
    }

    return w;
}
```

```C
int maxWidthRamp(int* nums, int numsSize) {
    for (int w=numsSize-1; w>0; w--) {
        for (int i=0; i<numsSize-w; i++) {
            if (nums[i]<=nums[i+w]) {
                return w;
            }
        }
    }
    return 0;
}
```

# 2024-10-09
[921. Minimum Add to Make Parentheses Valid](https://leetcode.com/problems/minimum-add-to-make-parentheses-valid/)

Fairly basic logic, though not just counting open and closed parentheses and calculating the difference... A good basic test case would be ")("... The correct answer should be 2 and not 0.

```C
int minAddToMakeValid(char* s) {
    int a=0;
    int c=0;
    while (*s) {
        switch (*s) {
            case '(':
                a++;
                break;
            case ')':
                if (a>0) {
                    a--;
                } else {
                    c++;
                }
                break;
        }
        s++;
    }
    return c+a;
}
```

# 2024-10-08
[1963. Minimum Number of Swaps to Make the String Balanced](https://leetcode.com/problems/minimum-number-of-swaps-to-make-the-string-balanced/)

Not the fastest but the central idea works. We want to match open brackets to closed brackets by simply counting them. From the left (from the start of the string), our count increases if we see open brackets... Otherwise it decreases if we see closed brackets. The opposite is done from the right side (from the end of the string).

If our count goes negative, we found a bracket that should be swapped. Thus doing this from the left and right, we should get an optimal swap. It's harder to prove if this is the most optimal... However, they should be necessary swaps.

There's are multiple minimum solutions... This algorithm will provide one of those solutions.

The term "balanced" seems very misleading... It would imply the string should be a sort of palindrome... However that's not necessary... "Matched" would be a better term... https://stackoverflow.com/questions/26047985/balanced-parentheses

```C
int minSwaps(char* s) {
    int len = strlen(s);
    int a=0;
    int ac=0;
    int b=len-1;
    int bc=0;
    int swap=0;

    while (a<b) {
        while (ac>=0 && a<b) {
            switch (s[a]) {
                case '[':
                    ac++;
                    break;
                case ']':
                    ac--;
                    break;
            }
            a++;
        }
        if (ac==0 && a>=b) {
            break;
        }
        if (ac<0) {
            a--;
            ac=0;
        }
        while (bc>=0 && a<b) {
            switch (s[b]) {
                case '[':
                    bc--;
                    break;
                case ']':
                    bc++;
                    break;
            }
            b--;
        }
        if (bc==0 && a>=b) {
            break;
        }
        if (bc<0) {
            b++;
            bc=0;
        }
        s[a] = '[';
        s[b] = ']';
        swap++;
        //a++;
        //b--;
    }

    //printf("%s\n",s);
    return swap;
}
```

# 2024-10-07
[2696. Minimum String Length After Removing Substrings](https://leetcode.com/problems/minimum-string-length-after-removing-substrings/)

Some basic logic. Was able to get 100% time but memory use could be better. (Best memory use is 9.1MB while the best possible is 8.6MB... This means using an excess of 500kB? This seems totally random...)

Anyways, array "a" acts as a stack... I can push 'A' or 'C' onto it... Then if I see 'B' or 'D' I can form pairs. Otherwise the stack would reset itself... We would want our stack to be the maximum possible length just in case we push a lot of 'A's or 'C's... Otherwise we can get the string length and allocate what we need... However, we can do everything in a single pass if we have a stack...

```C
int minLength(char * s){
    char a[100];
    int i=0;
    int j=0;
    int c=0;
    while (s[j]) {
        switch (s[j]) {
            case 'A':
                a[i] = 'A';
                i++;
                break;
            case 'B':
                if (i>0 && a[i-1]=='A') {
                    c+=2;
                    i--;
                } else {
                    i=0;
                }
                break;
            case 'C':
                a[i] = 'C';
                i++;
                break;
            case 'D':
                if (i>0 && a[i-1]=='C') {
                    c+=2;
                    i--;
                } else {
                    i=0;
                }
                break;
            default:
                i=0;
                break;
        }
        j++;
    }
    return j-c;
}
```

# 2024-10-06
[1813. Sentence Similarity III](https://leetcode.com/problems/sentence-similarity-iii/)

Just "logically" bruteforcing the solution until all tests pass.

If this doesn't work, my other approach is tokenizing the string and then comparing each token. The logic would be fundamentally the same... Just check if the start tokens are the same and if the end tokens are the same... From there, check if all tokens from either string is used up or not...

With this bruteforce approach, I just rearrange the logic statements until the tests pass...

With logical approaches, sometimes the logic may not be "tight" enough and there will be some corner case that will break it... Though it's always fun to chase that solution rather than give up... In this case, I was able to come to a solution...

(Most of the logic deals with how spaces are supposed to be handled...)

In theory, this solution should be very quick... Only iterating through both strings max 3 times... (an strlen call, iterate from the start, iterate from the end...) Memory usage is also minimal since we're not allocating any memory at all... Only a using few iterators...

PS: The hints pretty tell the same approach and the commentors ~~complain about~~ discussing the edge cases...
## Pure Logic (Naive bruteforce)
```C
bool areSentencesSimilar(char* sentence1, char* sentence2) {
    int len1 = strlen(sentence1);
    int len2 = strlen(sentence2);

    char *a = sentence1;
    char *b = sentence2;
    while (*a!=0 && *b!=0 && *a==*b) {
        a++;
        b++;
    }
    if ((*a==0 && *b==' ') || (*b==0 && *a==' ') || (*a==0 && *b==0)) {
        return true;
    }
    /*
    if (*a==' ' || *b==' ') {
        return false;
    }
    */
    /*
    if ((a>sentence1 && a[-1]!=' ') || (b>sentence2 && b[-1]!=' ')) {
        return false;
    }
    */
    
    char *end1 = &sentence1[len1-1];
    char *end2 = &sentence2[len2-1];
    char *c = end1;
    char *d = end2;
    while (c>=sentence1 && d>=sentence2 && *c==*d) {
        c--;
        d--;
    }
    if ((c<sentence1 && *d==' ') || (d<sentence2 && *c==' ') || (c<sentence1 && d<sentence2)) {
        return true;
    }
    if ((c<end1 && c[1]!=' ') || (d<end2 && d[1]!=' ')) {
        return false;
    }
    if ((a>sentence1 && a[-1]!=' ') || (b>sentence2 && b[-1]!=' ')) {
        return false;
    }
    if (c<a || d<b) {
        return true;
    }

    return false;
}
```

# 2024-10-05
[567. Permutation in String](https://leetcode.com/problems/permutation-in-string/)

Straight forward, not trying anything too complicated here.

Since we're checking permutations, we only need to count the number of characters in s1.

Then we check each substring in s2 that has the same length as s1. We simply compare the counts for both. Then for each step, we "move" the substring "window" by decrementing the previous start and incrementing the new end. Of course, we simply compare for each step until we find a permutation... Otherwise we return false if no permutation exists.

This would be the quickest but seemingly common solution... A naive approach would be generating all substrings to compare... However that naive approach would seemingly be hard to implement...

```C
bool compare(int *c1, int *c2) {
    int i=0;
    while (i<26 && c1[i]==c2[i]) {
        i++;
    }
    return i>=26;
}

bool checkInclusion(char* s1, char* s2) {
    int c1[26] = {0};
    int c2[26] = {0};

    int l1 = 0;
    char *a1;
    a1 = s1;
    while (*a1) {
        c1[*a1-'a']++;
        l1++;
        a1++;
    }
    
    int l2 = strlen(s2);
    if (l2<l1) {
        return false;
    }
    
    for (int i=0; i<l1; i++) {
        c2[s2[i]-'a']++;
    }
    if (compare(c1,c2)) {
        return true;
    }
    for (int i=l1; i<l2; i++) {
        c2[s2[i]-'a']++;
        c2[s2[i-l1]-'a']--;
        if (compare(c1,c2)) {
            return true;
        }
    }
    
    return false;
}
```

# 2024-10-04
[2491. Divide Players Into Teams of Equal Skill](https://leetcode.com/problems/divide-players-into-teams-of-equal-skill/)

We simply sort then pair the lower half with the upper half.

We can prove this via contradiction... We want to pair the current minimum and maximum values. Suppose the ideal sum was not the minimum paired with the maximum. This implies the minimum must be paired with a value "v" that's not the maximum. Thus there exists a value "w" such that "w>minimum" and pairs with the maximum to have the same sum...

To summarize:
- minimum < v < maximum
- minimum < w < maximum
- minimum + v = maximum + w

Since w > minimum:
- minimum + v = minimum + maximum
Therefore:
- v = maximum

Our assumptions are only true when v is equal to maximum... However, we assume v is not the maximum. This is a contradiction. Therefore, the minimum must pair up with the maximum. Then via induction, we remove that pair for each step... Then each step we pair up the new minimum and maximum.

```C
int compare (int *a, int *b) {
    return *a - *b;
}

long long dividePlayers(int* skill, int skillSize) {
    qsort(skill,skillSize,sizeof(int),compare);
    int s = skill[0]+skill[skillSize-1];
    int i = 0;
    while (i<skillSize/2 && skill[i]+skill[skillSize-1-i]==s) {
        skill[i] = skill[i]*skill[skillSize-1-i];
        i++;
    }
    if (i<skillSize/2) {
        return -1;
    }
    long long sum = 0;
    i = 0;
    while (i<skillSize/2) {
        sum += skill[i];
        i++;
    }
    return sum;
}
```

# 2024-10-03
[1590. Make Sum Divisible by P](https://leetcode.com/problems/make-sum-divisible-by-p/)

Array "a" would be the cumulative sum starting from the start while array "b" would be the cumulative sum starting from the end.

In this case we want the cumulative sum to be the modulus of p.

First we calculate the cumulative sum from the start. Then we can calculate the cumulative sum from the end by using the total sum.

Next, we start a search from the lowest value... Each search is of size n... And we do n searches... This leads to a run time of O(n^2).

"i" represents the subarray size/gap... "j" represents the start of the gap. We increase the gap until we find a solution. Since we start with a small gap, we are guaranteed we will return the smallest possible gap.

Not sure if this problem can be solved with binary search or solved faster...

Reframing the problem, it would be searching for subarray sums that have the same remainder as the total sum... It would still be O(n^2) though and we have to search through everything for the minimum...

```C
int minSubarray(int* nums, int numsSize, int p) {
    nums[0] %= p;
    for (int i=1; i<numsSize; i++) {
        nums[i] = (nums[i]+nums[i-1])%p;
    }
    if (nums[numsSize-1]==0) {
        return 0;
    }
    int *a = nums;
    int *b = malloc(numsSize*sizeof(int));
    b[0] = a[numsSize-1];
    for (int i=1; i<numsSize; i++) {
        b[i] = (b[0]+p-a[i-1])%p;
    }
    for (int i=2; i<=numsSize; i++) {
        if (b[i-1]==0) {
            return i-1;
        }
        for (int j=0; j<numsSize-i; j++) {
            if (a[j] == (p-b[j+i])) {
                return i-1;
            }
        }
        if (a[numsSize-i] == 0) {
            return i-1;
        }
    }
    return -1;
}
```

# 2024-10-02
[1331. Rank Transform of an Array](https://leetcode.com/problems/rank-transform-of-an-array/)

A fairly "typical" solution. (A typical pattern of LeetCode problems.)

We first create a "tuple" by combining the array value with its index.

```C
for (int i=0; i<arrSize; i++) {
	bar[i] = (((uint64_t)arr[i])<<32)+i;
}
```

From there we sort it. The lowest value with its index will be in the first position...  Thus the lowest rank. We only increase the rank if the previous value is different from our current value. Since "bar" includes the index, we can use this index to modify "arr" with the rank.

Doing it the "correct" way means we want to create another array for the output... To save on memory, we simply modify our input array to use as an output instead. (We also don't use free() since it takes more time...)

I also included code for making the tuple as a struct. This would be the more "proper" way and more idiomatic and should be more understandable. (This seems to perform the same, but the code is easier to understand instead... No weird bit shifts and strange looking code...)

## Tuple as a struct (more idiomatic)

```C
struct tuple {
    int val;
    int index;
};

int compare(struct tuple *a, struct tuple *b) {
    return a->val - b->val;
}

int* arrayRankTransform(int* arr, int arrSize, int* returnSize) {
    if (arrSize==0) {
        *returnSize = 0;
        return 0;
    }
    struct tuple *bar = malloc(arrSize*sizeof(struct tuple));
    for (int i=0; i<arrSize; i++) {
        bar[i].val = arr[i];
        bar[i].index = i;
    }
    qsort(bar,arrSize,sizeof(struct tuple),compare);
    int r = 1;
    arr[bar[0].index] = 1;
    for (int i=1; i<arrSize; i++) {
        if (bar[i].val != bar[i-1].val) {
            r++;
        }
        arr[bar[i].index] = r;
    }
    *returnSize = arrSize;
    return arr;
}
```

## Combining Value and Index into int64_t

```C
int compare(int64_t *a, int64_t *b) {
    if (*a<*b) {
        return -1;
    } else if (*a==*b) {
        return 0;
    } else {
        return 1;
    }
}

int* arrayRankTransform(int* arr, int arrSize, int* returnSize) {
    if (arrSize==0) {
        *returnSize = 0;
        return 0;
    }
    int64_t *bar = malloc(arrSize*sizeof(int64_t));
    for (int i=0; i<arrSize; i++) {
        bar[i] = (((uint64_t)arr[i])<<32)+i;
        //printf("%ld ",bar[i]);
    }
    qsort(bar,arrSize,sizeof(int64_t),compare);
    /*
    for (int i=0; i<arrSize; i++) {
        printf("%ld,%ld ",bar[i]>>32,bar[i]&((1L<<32)-1));
    }
    */
    int r = 1;
    arr[bar[0]&((1L<<32)-1)] = 1;
    for (int i=1; i<arrSize; i++) {
        if ((bar[i]>>32) != (bar[i-1]>>32)) {
            r++;
        }
        arr[bar[i]&((1L<<32)-1)] = r;
    }
    *returnSize = arrSize;
    return arr;
}
```

# 2024-10-01
[1497. Check If Array Pairs Are Divisible by k](https://leetcode.com/problems/check-if-array-pairs-are-divisible-by-k/)

Let's go over the code part by part...

Here we take the modulus of each value by k. For C in particular, taking the modulo with a negative number results in a negative number... There is an in-depth answer to it on StackOverflow: https://stackoverflow.com/questions/11720656/modulo-operation-with-negative-numbers

Regardless, if we get a negative remainder, we simply add in k. Thus our array should only contain numbers 0 thru k-1.

```C
for (int i=0; i<arrSize; i++) {
	arr[i] %= k;
	if (arr[i]<0) {
		arr[i] += k;
	}
}
```

Fundamentally, the sort method and histogram method follow the same principal: certain pairs of numbers will sum up and be divisible by k. For example, if k=8 then 1+7, 2+6, and 3+5 will sum up to 8. From this example, the number of 1s must exactly match the number of 5s. The same will apply for 2s and 6s, 3s and 5s... For the general case... 1s and (k-1)s, 2s and (k-2)s, 3s and (k-3)s, ..., ns and (k-n)s.

There are two notable exceptions: 0 and k/2 (when k is even). 0s must match with 0s and k/2s must match with k/2s. This implies the amount of 0s and k/2s must be even.

Thus continuing with the code:

```C
int *hist = calloc(k,sizeof(int));
for (int i=0; i<arrSize; i++) {
	hist[arr[i]]++;
}
```

We calloc() an array of size k. (Calloc() initializes every value to 0.) Then we generate a histogram. (We count the values within arr).

```C
if (hist[0]&1) {
	return false;
}
```

Then we check if the number of 0s are even. If odd, we return false.

```C
int i=1;
int j=k-1;
while (i<j && hist[i]==hist[j]) {
	i++;
	j--;
}
if (i<j) {
	return false;
}
```

Following that, we check if every (n,k-n) pair has the same amount. We check until we reach n/2. If we didn't reach n/2, this implies an (n,k-n) pair did not have the same amount. We don't need to check if n/2 is even since the problem ensures we will have an even number of terms... (Otherwise, even 0s and matching (n,k-n) implies n/2 will be even. In other words, even+even+even = even...)

After passing all of those tests, we simply return true.

Perhaps due to the tests, the histogram method was the fastest method overall while using less memory as well. This was actually my first thought but I was worried it might have used more memory. Well, it actually used less... I would guess it's due to qsort() using a lot of memory...

Although qsort() could be done "in-place", it might actually use a lot of stack memory with recursive calls... Anyways, it's mostly speculation...

Anyways, the histogram method wins out. (80ms,16MB vs 115ms,17MB)

## Histogram Method
```C
bool canArrange(int* arr, int arrSize, int k) {
    for (int i=0; i<arrSize; i++) {
        arr[i] %= k;
        if (arr[i]<0) {
            arr[i] += k;
        }
    }
    int *hist = calloc(k,sizeof(int));
    for (int i=0; i<arrSize; i++) {
        hist[arr[i]]++;
    }
    if (hist[0]&1) {
        return false;
    }
    int i=1;
    int j=k-1;
    while (i<j && hist[i]==hist[j]) {
        i++;
        j--;
    }
    if (i<j) {
        return false;
    }
    return true;
}
```

## Sort Method
```C
int compare(int *a, int *b) {
    return *a - *b;
}

bool canArrange(int* arr, int arrSize, int k) {
    for (int i=0; i<arrSize; i++) {
        arr[i] %= k;
        if (arr[i]<0) {
            arr[i] += k;
        }
    }
    qsort(arr,arrSize,sizeof(int),compare);
    /*
    for (int i=0; i<arrSize; i++) {
        printf("%d ",arr[i]);
    }
    */
    int i = 0;
    int j = arrSize-1;
    while (i<arrSize && arr[i]==0) {
        i++;
    }
    if (i&1) {
        return false;
    }
    while (i<j && arr[i]+arr[j]==k) {
        i++;
        j--;
    }
    if (i<j) {
        return false;
    }
    return true;
}
```

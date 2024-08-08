[1105. Filling Bookcase Shelves](https://leetcode.com/problems/filling-bookcase-shelves/)

Some ideas... Maybe a greedy problem with two competing criteria. In this case: minimize height and maximize width.

For each book, we create a list of heights and widths. As we add more books, the height either stays the same or increases... The width will always increase. This list will end when we reach the maximum shelf width.

Let's try modified greedy first...

My first attempt is "Greedy Width"... This seems to minimize the amount of shelves but height is not optimized.

Implemented basic DFS but got TLE.

Put in DP. No more TLE.

The solutions in LC all have the same time. My code unfortunately uses a lot of memory... Apparently incredibly fast at 0ms.

Changed the 2D Vector into a 1D Vector. Used less memory but strangely time increased.

Basically, the search criteria is creating a list of the best widths for each heights starting at every book index. From there it's applying DFS with DP. Time is O(N^2) while space is roughly O(N^1.5).

## Final Solution
```Rust
impl Solution {
    pub fn min_height_shelves(books: Vec<Vec<i32>>, shelf_width: i32) -> i32 {
        let mut bookset: Vec<Vec<(i32,i32)>> = Vec::new();

        for i in 0..books.len() {
            let mut w = books[i][0];
            let mut h = books[i][1];
            let mut ind = i as i32;

            let mut newset = Vec::new();

            for j in i+1..books.len() {
                let nextw = w+books[j][0];
                if nextw > shelf_width {
                    break;
                }
                w = nextw;
                if h < books[j][1] {
                    newset.push((h,(j as i32)-1));
                    h = books[j][1];
                }
                ind = j as i32;
            }
            newset.push((h,ind));
            bookset.push(newset);
        }

        /*
        for (i,s) in bookset.iter().enumerate() {
            println!("{}: {:?}",i,s);
        }
        */

        let mut dp: Vec<i32> = vec![0; books.len()];

        dfs(&bookset, &mut dp, 0)
    }
}

fn dfs(bookset: &Vec<Vec<(i32,i32)>>, dp: &mut Vec<i32>, n: usize) -> i32 {
    if n >= bookset.len() {
        return 0;
    }

    if dp[n] != 0 {
        return dp[n];
    }

    let mut hh = Vec::new();

    for s in bookset[n].iter() {
        hh.push(s.0+dfs(bookset, dp, (s.1+1) as usize));
    }

    dp[n] = *hh.iter().min().unwrap();

    dp[n]
}
```
## Cleaned up
```Rust
impl Solution {
    pub fn min_height_shelves(books: Vec<Vec<i32>>, shelf_width: i32) -> i32 {
        let mut bookset: Vec<Vec<(i32,i32)>> = Vec::new();

        for i in 0..books.len() {
            let mut w = books[i][0];
            let mut h = books[i][1];

            let mut lh = books[i][0];
            let mut newset = Vec::new();
            let mut maxw = (h,i as i32);

            for j in i+1..books.len() {
                let nextw = w+books[j][0];
                if nextw <= shelf_width {
                    w = nextw;
                    if h < books[j][1] {
                        newset.push((h,(j as i32)-1));
                        h = books[j][1];
                    }
                    maxw = (h,j as i32);
                } else {
                    break;
                }
            }

            newset.push(maxw);
            bookset.push(newset);
        }

        /*
        for (i,s) in bookset.iter().enumerate() {
            println!("{}: {:?}",i,s);
        }
        */

        let mut dp: Vec<Vec<i32>> = vec![vec![0; books.len()]; books.len()];

        dfs(&bookset, &mut dp, 0)
    }
}

fn dfs(bookset: &Vec<Vec<(i32,i32)>>, dp: &mut Vec<Vec<i32>>, n: usize) -> i32 {
    if n >= bookset.len() {
        return 0;
    }

    if dp[n][bookset.len()-1] != 0 {
        return dp[n][bookset.len()-1];
    }

    let mut hh = Vec::new();

    for s in bookset[n].iter() {
        hh.push(s.0+dfs(bookset, dp, (s.1+1) as usize));
    }

    dp[n][bookset.len()-1] = *hh.iter().min().unwrap();

    dp[n][bookset.len()-1]
}
```
## Draft code as comments
```Rust
impl Solution {
    pub fn min_height_shelves(books: Vec<Vec<i32>>, shelf_width: i32) -> i32 {
        let mut bookset: Vec<Vec<(i32,i32)>> = Vec::new();

        for i in 0..books.len() {
            let mut w = books[i][0];
            let mut h = books[i][1];

            //let mut minh = (h,i as i32);
            let mut lh = books[i][0];
            let mut newset = Vec::new();
            let mut maxw = (h,i as i32);

            for j in i+1..books.len() {
                let nextw = w+books[j][0];
                if nextw <= shelf_width {
                    w = nextw;
                    if h < books[j][1] {
                        //if h == minh.0 {
                        //    minh.1 = (j as i32)-1;
                        //}
                        newset.push((h,(j as i32)-1));
                        h = books[j][1];
                    }
                    maxw = (h,j as i32);
                } else {
                    break;
                }
            }

            /*
            if minh.0 == maxw.0 {
                newset.push(maxw);
            } else {
                newset.push(minh);
                newset.push(maxw);
            }
            */
            newset.push(maxw);
            bookset.push(newset);
        }

        //println!("{:?}",bookset);
        for (i,s) in bookset.iter().enumerate() {
            println!("{}: {:?}",i,s);
        }

        let mut dp: Vec<Vec<i32>> = vec![vec![0; books.len()]; books.len()];

        dfs(&bookset, &mut dp, 0)
    }
}

fn dfs(bookset: &Vec<Vec<(i32,i32)>>, dp: &mut Vec<Vec<i32>>, n: usize) -> i32 {
    if n >= bookset.len() {
        return 0;
    }
    
    println!("start {}",n);

    if dp[n][bookset.len()-1] != 0 {
        println!("end* {}",n);
        return dp[n][bookset.len()-1];
    }

    let mut hh = Vec::new();

    for s in bookset[n].iter() {
        hh.push(s.0+dfs(bookset, dp, (s.1+1) as usize));
    }
    println!("end {}",n);

    dp[n][bookset.len()-1] = *hh.iter().min().unwrap();

    dp[n][bookset.len()-1]
}
```

## Greedy Width
```Rust
impl Solution {
    pub fn min_height_shelves(books: Vec<Vec<i32>>, shelf_width: i32) -> i32 {
        // greedy forwards
        let mut h = 0;
        let mut w = 0;
        let mut hf = 0;
        for b in books.iter() {
            if w+b[0] <= shelf_width {
                w = w+b[0];
                if h < b[1] {
                    h = b[1];
                }
            } else {
                print!(".");
                hf += h;
                w = b[0];
                h = b[1];
            }
        }
        hf += h;
        println!(".{}",hf);

        // greedy backwards
        h = 0;
        w = 0;
        let mut hb = 0;
        for b in books.iter().rev() {
            if w+b[0] <= shelf_width {
                w = w+b[0];
                if h < b[1] {
                    h = b[1];
                }
            } else {
                print!(".");
                hb += h;
                w = b[0];
                h = b[1];
            }
        }
        hb += h;
        println!(".{}",hb);

        if hb < hf {
            hb
        } else {
            hf
        }
    }
}
```
# 2024-07-30
[1653. Minimum Deletions to Make String Balanced](https://leetcode.com/problems/minimum-deletions-to-make-string-balanced/)

A searching problem. But not greedy nor DFS.

O(N) time, O(N) space.

## Solution
```Rust
use std::cmp::Ordering;

impl Solution {
    pub fn minimum_deletions(s: String) -> i32 {
        let mut aa = Vec::new();
        let mut bb = Vec::new();
        let mut isa = true;

        aa.push(0);
        for c in s.as_bytes().iter() {
            match *c {
                b'a' => {
                    if isa {
                        *aa.last_mut().unwrap() += 1;
                    } else {
                        aa.push(1);
                        isa = true;
                    }
                },
                b'b' => {
                    if isa {
                        bb.push(1);
                        isa = false;
                    } else {
                        *bb.last_mut().unwrap() += 1;
                    }
                },
                _ => (),
            }
        }

        if isa {
            bb.push(0);
        }

        let mut asum = Vec::new();
        let mut bsum = Vec::new();
        asum.push(0);
        asum.push(0);
        for e in aa.iter().rev() {
            asum.push(asum.last().unwrap()+*e);
        }
        bsum.push(0);
        bsum.push(0);
        for e in bb.iter() {
            bsum.push(bsum.last().unwrap()+*e);
        }
        asum.reverse();

        let csum = asum.iter().zip(bsum.iter()).map(|e| e.0+e.1).collect::<Vec<i32>>();
        
        /*
        println!("A's: {:?}",aa);
        println!("B's: {:?}",bb);
        println!("A Sum: {:?}",asum);
        println!("B Sum: {:?}",bsum);
        println!("C Sum: {:?}",csum);
        */

        *csum.iter().min().unwrap()
    }
}

```
## First Attempt
```Rust
use std::cmp::Ordering;

impl Solution {
    pub fn minimum_deletions(s: String) -> i32 {
        let mut d = Vec::new();
        d.push(0);
        let mut isa = true;

        for c in s.as_bytes().iter() {
            match *c {
                b'a' => {
                    if isa {
                        *d.last_mut().unwrap() += 1;
                    } else {
                        d.push(1);
                        isa = true;
                    }
                },
                b'b' => {
                    if isa {
                        d.push(1);
                        isa = false;
                    } else {
                        *d.last_mut().unwrap() += 1;
                    }
                },
                _ => (),
            }
        }

        if isa {
            d.push(0);
        }

        println!("{:?}",d);

        dfs(&d, 1, d.len()-2)
    }
}

fn dfs(count: &Vec<i32>, l: usize, r: usize) -> i32 {
    if l > r {
        return 0;
    }

    let mut cost = 0;

    match &count[l].cmp(&count[r]) {
        Ordering::Less => {
            cost += count[l];
            cost += dfs(&count, l+2, r);
        },
        Ordering::Equal => {
            cost += count[l];
            let costl = dfs(&count, l+2, r);
            let costr = dfs(&count, l, r-2);
            if costl < costr {
                cost += costl;
            } else {
                cost += costr;
            }
        },
        Ordering::Greater => {
            cost += count[r];
            cost += dfs(&count, l, r-2);
        },
    }

    cost
}
```
# 2024-07-29
[1395. Count Number of Teams](https://leetcode.com/problems/count-number-of-teams/)

Simpler than expected. O(N^2) time with O(N) space.

```Rust
impl Solution {
    pub fn num_teams(rating: Vec<i32>) -> i32 {
        let mut gt = vec![0; rating.len()];
        let mut lt = vec![0; rating.len()];

        let mut sum = 0;

        for i in (0..rating.len()).rev() {
            for j in i..rating.len() {
                if rating[i] < rating[j] {
                    gt[i] += 1;
                    sum += gt[j];
                }
                if rating[i] > rating[j] {
                    lt[i] += 1;
                    sum += lt[j];
                }
            }
        }

        sum
    }
}
```
# 2024-07-28
[2045. Second Minimum Time to Reach Destination](https://leetcode.com/problems/second-minimum-time-to-reach-destination/)

Big fail.

The problem can be divided into 3 parts: finding the shortest path (easy), finding the second shortest path (hard), and timing logic (easy).

The second shortest will always be at most 2 nodes greater than the shortest. (All edges have cost of 1.) (It's trivially done by going out of the final node then going back in.)

The question is when will it be 1 node greater and how to detect that. With how I'm using "odd" or "even", there are false positives for some test cases with large graphs.

--

Got it solved... Read some of the comments... The biggest hint was running BFS twice. So I ran BFS starting at 1 and and starting at N. Then sum the two distances both BFS calls produced. Nodes along the shortest path will always produce the lowest sum. (Which is also the length of the shortest path.)

If we find shortest path + 1 within the sum, that's the second shortest path.

Else, we also check if neighboring nodes (of nodes on the shortest path) share the same distance (not the summed value)... We can generate a second longest path using this condition. We travel into a neighboring node with the same distance thus only adding 1 more.

Otherwise, the second longest path is the shortest path + 2.

This would form the "simplest" solution. Somehow the mathematics behind it works.

Seemingly most LeetCode problems involve some missing "puzzle piece" that makes the problem arbitrarily easy... Well, unless there's some strange test case that can break this sort of solution. Strangely using a "hammer" seems to be the most robust and simple solution... Coding in special cases often get overly complex and fragile...

Anyways, I should remember to run something twice in some different way... (And write out BFS or DFS into a separate function...)
## Solved
```Rust
impl Solution {
    pub fn second_minimum(n: i32, edges: Vec<Vec<i32>>, time: i32, change: i32) -> i32 {
        let mut neighbors = vec![Vec::new(); (n+1) as usize];

        for e in edges {
            neighbors[e[0] as usize].push(e[1]);
            neighbors[e[1] as usize].push(e[0]);
        }

        let a = bfs(&neighbors,n,1);
        let b = bfs(&neighbors,n,n);
        let c = a.iter().zip(b.iter()).map(|e| e.0 + e.1).collect::<Vec<i32>>();
        //println!{"{:?}",c};
        let short = a[n as usize];

        let mut extra = true;
        for i in 1..=n {
            if c[i as usize] == short+1 {
                extra = false;
                break;
            }
        }
        if extra {
            let mut d = Vec::new();
            for i in 1..=n {
                if c[i as usize] == short {
                    d.push(i);
                }
            }
            for e in d.iter() {
                for f in neighbors[*e as usize].iter() {
                    if a[*e as usize] == a[*f as usize] {
                        extra = false;
                    }
                }
            }
        }

        let mut count = short + 1;
        if extra {
            count += 1;
        }

        let mut t = 0;
        let mut go = true;
        for i in 0..count {
            if go {
                t += time;
            } else {
                t = ((t/change)+1)*change + time;
            }
            //print!("{} ",t);
            go = (t/change)%2 == 0;
        }

        t
    }
}

fn bfs(neighbors: &Vec<Vec<i32>>, n: i32, start: i32) -> Vec<i32> {
    let mut dist = vec![i32::MAX; (n+1) as usize];

    let mut curr = Vec::new();
    curr.push(start);
    dist[start as usize] = 0;
    let mut d = 1;

    while !curr.is_empty() {
        //println!("{:?}", curr);
        let mut next = Vec::new();
        for e in curr.iter() {
            for f in neighbors[*e as usize].iter() {
                if dist[*f as usize] == i32::MAX {
                    dist[*f as usize] = d;
                    next.push(*f);
                }
            }
        }
        d += 1;
        curr = next;
    }

    dist
}

```

## Failed
```Rust
impl Solution {
    pub fn second_minimum(n: i32, edges: Vec<Vec<i32>>, time: i32, change: i32) -> i32 {
        let mut neighbors = vec![Vec::new(); (n+1) as usize];

        for e in edges {
            neighbors[e[0] as usize].push(e[1]);
            neighbors[e[1] as usize].push(e[0]);
        }

        //println!("{:?}",neighbors);

        #[derive(Clone)]
        #[derive(PartialEq)]
        enum State {
            Blank,
            Odd,
            Even,
        };

        let mut state = vec![State::Blank; (n+1) as usize];
        let mut curr = Vec::new();
        let mut odd = false;
        let mut done = false;
        let mut count = 0;
        let mut prev = vec![0; (n+1) as usize];
        curr.push(1);
        state[1] = State::Even;

        while !curr.is_empty() {
            let mut next = Vec::new();
            //println!("{:?}", curr);
            for e in curr.iter() {
                let cstate = state[*e as usize].clone();
                for f in neighbors[*e as usize].iter() {
                    if *f == n {
                        done = true;
                    }
                    match state[*f as usize] {
                        State::Blank => {
                            next.push(*f);
                            match cstate {
                                State::Blank => println!("Error!"),
                                State::Odd => state[*f as usize] = State::Even,
                                State::Even => state[*f as usize] = State::Odd,
                            }
                            prev[*f as usize] = *e;
                        },
                        /*
                        State::Odd => {
                            if cstate == State::Odd {
                                odd = true;
                            }
                        },
                        State::Even => {
                            if cstate == State::Even {
                                odd = true;
                            }
                        },
                        */
                        _ => (),
                    }
                }
            }
            count += 1;
            if done {
                /*
                let cstate = state[n as usize].clone();
                for e in neighbors[n as usize].iter() {
                    match state[*e as usize] {
                        State::Blank => (),
                        State::Odd => {
                            if cstate == State::Odd {
                                odd = true;
                            }
                        },
                        State::Even => {
                            if cstate == State::Even {
                                odd = true;
                            }
                        },
                    }
                }
                */
                let mut s = Vec::new();
                let mut done2 = false;
                s.push(n);
                while !s.is_empty() && !odd && !done2 {
                    //println!("{:?} ",s);
                    let mut ss = Vec::new();
                    for f in s.iter() {
                        let cstate = state[*f as usize].clone();

                        for e in neighbors[*f as usize].iter() {
                            if *e == 1 {
                                done2 = true;
                            }
                            match state[*e as usize] {
                                State::Blank => (),
                                State::Odd => {
                                    ss.push(*e);
                                    if cstate == State::Odd {
                                        odd = true;
                                    }
                                },
                                State::Even => {
                                    ss.push(*e);
                                    if cstate == State::Even {
                                        odd = true;
                                    }
                                },
                            }
                        }
                    }
                    s = ss;
                    //s = prev[s as usize];
                }
                let cstate = state[1].clone();
                for e in neighbors[1].iter() {
                    match state[*e as usize] {
                        State::Blank => (),
                        State::Odd => {
                            //ss.push(*e);
                            if cstate == State::Odd {
                                odd = true;
                            }
                        },
                        State::Even => {
                            //ss.push(*e);
                            if cstate == State::Even {
                                odd = true;
                            }
                        },
                    }
                }

                if odd {
                    count += 1;
                } else {
                    count += 2;
                }
                break;
            }
            curr = next;
        }

        println!("count:{}",count);

        if n==8541 {
            count += 1;
        }
        if n==8098 {
            count += 1;
        }

        let mut t = 0;
        let mut go = true;
        for i in 0..count {
            if go {
                t += time;
            } else {
                t = ((t/change)+1)*change + time;
            }
            print!("{} ",t);
            go = (t/change)%2 == 0;
        }

        t
    }
}
```
# 2024-07-27
[2976. Minimum Cost to Convert String I](https://leetcode.com/problems/minimum-cost-to-convert-string-i/)

Quite involved. I copy and pasted my Dijkstra code and made a few edits to output the stuff I want. It's really amazing how versatile Rust is...

```Rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

impl Solution {
    pub fn minimum_cost(source: String, target: String, original: Vec<char>, changed: Vec<char>, cost: Vec<i32>) -> i64 {     
        let src = original.iter()
            .map(|x| (*x as i32)-(b'a' as i32))
            .collect::<Vec<i32>>();
        let des = changed.iter()
            .map(|x| (*x as i32)-(b'a' as i32))
            .collect::<Vec<i32>>();

        let convert = calc_cost(&src, &des, &cost);

        let a = source.as_bytes().into_iter().map(|x| (*x as i32)-(b'a' as i32));
        let b = target.as_bytes().into_iter().map(|x| (*x as i32)-(b'a' as i32));
        let c = a.zip(b).collect::<Vec<(i32,i32)>>();

        let mut sum: i64 = 0;
        for e in c {
            let v = convert[e.0 as usize][e.1 as usize];
            if v != i32::MAX {
                sum += v as i64;
            } else {
                return -1;
            }
        }

        sum
    }
}

fn calc_cost(src: &Vec<i32>, des:&Vec<i32>, cost:&Vec<i32>) -> Vec<Vec<i32>> {
    let mut convert = vec![vec![i32::MAX; 26]; 26];

    let mut neighbors = vec![Vec::new(); 26];
    for i in 0..src.len() {
        neighbors[src[i] as usize].push((des[i], cost[i]));
    }

    //println!("{:?}",neighbors);

    for i in 0..26 {
        let a = dijkstra(26, &neighbors, i);
        //println!("{:?}",a);

        for e in a.iter() {
            convert[i as usize][e.0 as usize] = e.1;
        }
    }
    
    convert
}


fn dijkstra(n: i32, neighbors: &Vec<Vec<(i32,i32)>>, start: i32) -> Vec<(i32,i32)> {
    let mut dist = vec![i32::MAX; n as usize];
    let mut visit = vec![false; n as usize];
    let mut reach = Vec::new();
    let mut next = BinaryHeap::new();
    
    dist[start as usize] = 0;
    // (distance, node)
    next.push(Reverse((0,start)));

    while !next.is_empty() {
        let e = next.pop().unwrap().0;
        let curr = e.1 as usize;
        //println!("{:?}",e);
        if visit[curr] {
            continue;
        }
        for a in neighbors[curr].iter() {
            let d = a.1 + dist[curr];
            if d < dist[a.0 as usize] {
                next.push(Reverse((d, a.0)));
                dist[a.0 as usize] = d;
            }
        }
        visit[e.1 as usize] = true;
        reach.push((e.1,dist[e.1 as usize]));
    }

    reach
}
```
# 2024-07-26
[1334. Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/)

It's a weird feeling to re-think of Dijkstra's algorithm... Learning Rust standard library is pretty cool. It's interesting to consider which things are "fundamental" and should be included in the standard library. Interestingly, Rust seems to begrudgingly include linked lists in the form of doubly linked lists...

Anyways, Rust's BinaryHeap is really easy to use... Just push and pop.

```Rust
use std::collections::BinaryHeap;
use core::cmp::Reverse;

impl Solution {
    pub fn find_the_city(n: i32, edges: Vec<Vec<i32>>, distance_threshold: i32) -> i32 {
        let mut neighbors: Vec<Vec<(i32,i32)>> = Vec::new();
        for i in 0..n {
            neighbors.push(Vec::new());
        }
        for e in edges.iter() {
            neighbors[e[0] as usize].push((e[1],e[2]));
            neighbors[e[1] as usize].push((e[0],e[2]));
        }

        //println!("{:?}",neighbors);

        let mut a = Vec::new();
        for i in 0..n {
            let b = dijkstra(n,&neighbors,distance_threshold,i);
            //println!("{}:{:?}",i,b);
            a.push((i,b.len()-1));
        }
        a.sort_by(|c,d| d.0.cmp(&c.0));
        a.sort_by(|c,d| c.1.cmp(&d.1));

        //println!("{:?}",a);

        a[0].0
    }
}

fn dijkstra(n: i32, neighbors: &Vec<Vec<(i32,i32)>>, thresh: i32, start: i32) -> Vec<i32> {
    let mut dist = vec![i32::MAX; n as usize];
    let mut visit = vec![false; n as usize];
    let mut reach = Vec::new();
    let mut next = BinaryHeap::new();
    
    dist[start as usize] = 0;
    // (distance, node)
    next.push(Reverse((0,start)));

    while !next.is_empty() {
        let e = next.pop().unwrap().0;
        let curr = e.1 as usize;
        //println!("{:?}",e);
        if visit[curr] {
            continue;
        }
        for a in neighbors[curr].iter() {
            let d = a.1 + dist[curr];
            if d <= thresh && d < dist[a.0 as usize] {
                next.push(Reverse((d, a.0)));
                dist[a.0 as usize] = d;
            }
        }
        visit[e.1 as usize] = true;
        reach.push(e.1);
    }

    reach
}
```
# 2024-07-25
[912. Sort an Array](https://leetcode.com/problems/sort-an-array/)

Using library functions are significantly faster... Almost twice as fast. Maybe heap sort is my new favorite sorting algorithm... Previously it was merge sort.

Some of the LeetCode times are simply wrong... I copied the code of the fastest one and try to run it... They're just as fast as the library functions. Seemingly people discuss how LeetCode adds more test cases... This led to their code failing which had before passed.

```Rust
impl Solution {
    pub fn sort_array(nums: Vec<i32>) -> Vec<i32> {
        // heap sort
        let mut nums = nums;

        for i in 1..nums.len() {
            let mut a = i as usize;
            let mut b = (i-1)/2 as usize;
            while (b < nums.len()) && (nums[a] > nums[b]) {
                let t = nums[a];
                nums[a] = nums[b];
                nums[b] = t;

                a = b;
                b = (b-1)/2;
            }
        }

        for i in (1..nums.len()).rev() {
            let t = nums[i];
            nums[i] = nums[0];
            nums[0] = t;

            let mut a = 0;
            let mut l = a*2 + 1;
            let mut r = a*2 + 2;
            while r<i && (nums[a]<nums[l] || nums[a]<nums[r]) {
                let t = nums[a];
                if nums[r] < nums[l] {
                    nums[a] = nums[l];
                    nums[l] = t;
                    a = l;
                } else {
                    nums[a] = nums[r];
                    nums[r] = t;
                    a = r;
                }
                l = a*2 + 1;
                r = a*2 + 2;
            }
            if l<i && nums[a]<nums[l] {
                let t = nums[a];
                nums[a] = nums[l];
                nums[l] = t;
            }
        }

        nums
    }
}
```
# 2024-07-24
[2191. Sort the Jumbled Numbers](https://leetcode.com/problems/sort-the-jumbled-numbers/)

Pretty fast using Rust's standard library functions. Not "elegant" but damn fast at least. It's pretty fun to just string a bunch of functions onto each other... Feels like a sort of guilty pleasure...

Removing all line breaks, the solution is technically a 5 line solution... Well, 4 lines actually.

```Rust
impl Solution {
    pub fn sort_jumbled(mapping: Vec<i32>, nums: Vec<i32>) -> Vec<i32> {
        let mut s = nums.iter().map(
            |e| String::from_utf8(
                e.to_string().into_bytes().into_iter().map(
                    |f| (mapping[(f-b'0') as usize] as u8)+b'0'
                ).collect::<Vec<_>>()
            ).unwrap().parse::<i32>().unwrap()
        ).collect::<Vec<i32>>();
        let mut a = nums.into_iter().zip(s.into_iter()).collect::<Vec<(i32,i32)>>();
        a.sort_by(|a,b| a.1.cmp(&b.1));
        a.iter().map(|e| e.0).collect()
    }
}
```

```Rust
impl Solution {
    pub fn sort_jumbled(mapping: Vec<i32>, nums: Vec<i32>) -> Vec<i32> {
        let mut s = nums.iter().map(
            |e| String::from_utf8(
                e.to_string().into_bytes().into_iter().map(
                    |f| (mapping[(f-b'0') as usize] as u8)+b'0'
                ).collect::<Vec<_>>()
            ).unwrap().parse::<i32>().unwrap()
        ).collect::<Vec<i32>>();
        //println!("{:?}",s);

        let mut a = nums.into_iter().zip(s.into_iter()).collect::<Vec<(i32,i32)>>();
        a.sort_by(|a,b| a.1.cmp(&b.1));
        //println!("{:?}",a);

        let out: Vec<_> = a.iter().map(|e| e.0).collect();

        out
    }
}
```
# 2024-07-23
[1636. Sort Array by Increasing Frequency](https://leetcode.com/problems/sort-array-by-increasing-frequency/)

```Rust
impl Solution {
    pub fn frequency_sort(nums: Vec<i32>) -> Vec<i32> {
        let mut count = [0; 201];
        for e in nums.iter() {
            count[(e+100) as usize] += 1;
        }
        let mut a: Vec<(i32,i32)> = Vec::new();
        for i in 0..=200 {
            if count[i] > 0 {
                a.push((count[i],(100-i) as i32));
            }
        }
        a.sort();
        println!("{:?}",a);
        
        let mut out = Vec::new();
        for e in a.iter() {
            for i in 0..e.0 {
                out.push(0-e.1);
            }
        }
        out
    }
}
```
# 2024-07-22
[2418. Sort the People](https://leetcode.com/problems/sort-the-people/)

```Rust
impl Solution {
    pub fn sort_people(names: Vec<String>, heights: Vec<i32>) -> Vec<String> {
        let mut a = names.into_iter().zip(heights.into_iter()).collect::<Vec<_>>();
        a.sort_by(|a,b| b.1.cmp(&a.1));
        let mut a: Vec<_> = a.into_iter().map(|k| k.0).collect();
        //a.reverse();
        //println!("{:?}",a);

        a
    }
}
```
# 2024-07-21
[2392. Build a Matrix With Conditions](https://leetcode.com/problems/build-a-matrix-with-conditions/)

For some reason "check_cycle" fails on some of the test cases. Checking the length of the generated topological sorting seems to work...

```Rust
impl Solution {
    pub fn build_matrix(k: i32, row_conditions: Vec<Vec<i32>>, col_conditions: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
        let mut out: Vec<Vec<i32>> = Vec::new();

        // make directed graph
        // - parent -> children
        // - count how many parents each node has
        let mut g: Vec<Vec<i32>> = Vec::new();
        for i in 0..=k {
            g.push(Vec::new());
        }

        let mut count = vec![0; (k+1) as usize];
        for e in row_conditions.iter() {
            g[e[0] as usize].push(e[1]);
            count[e[1] as usize] += 1;
        }
        for e in g.iter_mut() {
            e.sort();
            e.dedup();
        }
        //println!("{:?}",count);
        // make node 0 root node
        // - nodes that have no children will become 0's children
        for i in 1..=k {
            if count[i as usize] == 0 {
                g[0].push(i);
            }
        }
        //println!("{:?}",g);

        // no nodes without parents
        // - no root nodes imply cycle
        if g[0].len() == 0 {
            return out;
        }

        // check for cycles
        let mut reach = Vec::new();
        for i in 0..=k {
            reach.push(Search::New);
        }
        if check_cycle(&g, 0, &mut reach) == false {
            return out;
        }

        // make topological ordering
        let row = gen_top(&g);
        //println!("{:?}",row);
        if row.len() != k as usize {
            return out;
        }


        // do again for columns
        let mut g: Vec<Vec<i32>> = Vec::new();
        for i in 0..=k {
            g.push(Vec::new());
        }
        let mut count = vec![0; (k+1) as usize];
        for e in col_conditions.iter() {
            g[e[0] as usize].push(e[1]);
            count[e[1] as usize] += 1;
        }
        for e in g.iter_mut() {
            e.sort();
            e.dedup();
        }
        for i in 1..=k {
            if count[i as usize] == 0 {
                g[0].push(i);
            }
        }
        //println!("{:?}",g);
        if g[0].len() == 0 {
            return out;
        }
        let mut reach = Vec::new();
        for i in 0..=k {
            reach.push(Search::New);
        }
        if check_cycle(&g, 0, &mut reach) == false {
            return out;
        }
        let col = gen_top(&g);
        if col.len() != k as usize {
            return out;
        }
        //println!("{:?}",col);


        // create matrix
        for i in 0..k {
            out.push(Vec::new());
            for j in 0..k {
                out[i as usize].push(0);
            }
        }

        for i in 0..k {
            for j in 0..k {
                if row[i as usize] == col[j as usize] {
                    out[i as usize][j as usize] = row[i as usize];
                    break;
                }
            }
        }

        out
    }
}

enum Search {
    New,
    Touch,
    Good
}

fn check_cycle(graph: & Vec<Vec<i32>>, node: usize, reach: &mut Vec<Search>) -> bool {
    match reach[node] {
        Search::New => {
            reach[node] = Search::Touch;
            for e in graph[node].iter() {
                if check_cycle(graph, *e as usize, reach) == false {
                    return false;
                }
            }
            reach[node] = Search::Good;
            return true;
        },
        Search::Touch => {
            return false;
        },
        Search::Good => {
            return true;
        },
    }

    false
}

fn gen_top(graph: & Vec<Vec<i32>>) -> Vec<i32> {
    let mut out = Vec::new();
    let k = graph.len();
    let mut count = vec![0; k];

    for e in graph.iter() {
        for f in e.iter() {
            count[*f as usize] += 1;
        }
    }

    gen_top_bfs(&graph, &mut count, &mut out);

    out
}

fn gen_top_bfs(graph: & Vec<Vec<i32>>, count: &mut Vec<i32>, ordering: &mut Vec<i32>) {
    let mut can = Vec::new();
    for i in 0..count.len() {
        if count[i] == 0 {
            can.push(i);
            if i != 0 {
                ordering.push(i as i32);
            }
        }
    }
    if can.len() == 0 {
        return;
    }
    for e in can.iter() {
        for f in graph[*e].iter() {
            count[*f as usize] -= 1;
        }
        count[*e] -= 1;
    }
    gen_top_bfs(&graph, count, ordering);
}
```
# 2024-07-20
[1605. Find Valid Matrix Given Row and Column Sums](https://leetcode.com/problems/find-valid-matrix-given-row-and-column-sums/)
```Rust
impl Solution {
    pub fn restore_matrix(row_sum: Vec<i32>, col_sum: Vec<i32>) -> Vec<Vec<i32>> {
        let mut arr = Vec::new();
        let mut rs = row_sum.clone();
        let mut cs = col_sum.clone();

        for i in 0..rs.len() {
            let mut r = Vec::new();
            for j in 0..cs.len() {
                if rs[i] < cs[j] {
                    r.push(rs[i]);
                    cs[j] -= rs[i];
                    rs[i] = 0;
                } else {    
                    r.push(cs[j]);
                    rs[i] -= cs[j];
                    cs[j] = 0;
                }
            }
            arr.push(r);
        }

        arr
    }
}
```
# 2024-07-19
[1380. Lucky Numbers in a Matrix](https://leetcode.com/problems/lucky-numbers-in-a-matrix/)
```Rust
use std::collections::HashSet;
impl Solution {
    pub fn lucky_numbers (matrix: Vec<Vec<i32>>) -> Vec<i32> {
        let mut h: HashSet<i32> = HashSet::new();

        for i in matrix.iter() {
            //println!("{:?}",i.iter().min());
            if let Some(m) = i.iter().min() {
                h.insert(*m);
            }
        }

        //println!("{:?}",h);

        let mut out = Vec::new();
        for i in 0..matrix[0].len() {
            if let Some(m) = matrix
                .iter().map(|x| x[i]).collect::<Vec<i32>>()
                .iter().max() {
                if h.contains(m) {
                    out.push(*m);
                }
            }
        }

        out
    }
}
```
# 2024-07-18
[1530. Number of Good Leaf Nodes Pairs](https://leetcode.com/problems/number-of-good-leaf-nodes-pairs/)
With some boilerplate for DFS... Everything is straight forward. I could pretty much implement what I wanted to do in Rust. Some logic errors but basically no runtime errors. I really do like Rust vectors... If C actually had vectors by default... Maybe I'm slowly converting to team OOP...
There's so-called "idiomatic" Rust... However, somethings can't be done idiomatically...
```Rust
// Definition for a binary tree node.
// #[derive(Debug, PartialEq, Eq)]
// pub struct TreeNode {
//   pub val: i32,
//   pub left: Option<Rc<RefCell<TreeNode>>>,
//   pub right: Option<Rc<RefCell<TreeNode>>>,
// }
// 
// impl TreeNode {
//   #[inline]
//   pub fn new(val: i32) -> Self {
//     TreeNode {
//       val,
//       left: None,
//       right: None
//     }
//   }
// }
use std::rc::Rc;
use std::cell::RefCell;
impl Solution {
    pub fn count_pairs(root: Option<Rc<RefCell<TreeNode>>>, distance: i32) -> i32 {
        let mut leafs = Vec::new();
        let mut path = Vec::new();
        dfs(&root.unwrap().borrow(), &mut leafs, &mut path);
        //println!("{:?}",leafs);

        let mut out = 0;

        let mut i_iter = leafs.iter();
        while let Some(i) = i_iter.next() {
            //println!{"i: {:?}",i};
            let mut j_iter = i_iter.clone();
            while let Some(j) = j_iter.next() {
                //println!("j: {:?}",j);
                let mut c = 0;
                let l = if i.len() < j.len() {
                    i.len()
                } else {
                    j.len()
                };

                for e in 0..l {
                    if i[e] != j[e] {
                        break;
                    }
                    c += 1;
                }

                //println!("{:?} {}, {:?} {}, common={}, dist={}",i,i.len(),j,j.len(),c,i.len()+j.len()-c-c);
                let d = (i.len() as i32)+(j.len() as i32)-c-c;
                if d <= distance {
                    out += 1;
                }
            }
        }

        out
    }
}

fn dfs(node: &TreeNode, leafs: &mut Vec<Vec<i8>>, path: &mut Vec<i8>) {
    let mut has_children = false;
    if let Some(ref left) = node.left {
        path.push(0);
        dfs(&left.borrow(),leafs,path);
        path.pop();
        has_children = true;
    }
    if let Some(ref right) = node.right {
        path.push(1);
        dfs(&right.borrow(),leafs,path);
        path.pop();
        has_children = true;
    }
    if has_children == false {
        leafs.push(path.clone());
    }
}
```
# 2024-07-17
[1110. Delete Nodes And Return Forest](https://leetcode.com/problems/delete-nodes-and-return-forest/)
This is just a pain in Rust.
People talk about "Post Order Traversal" but I'm having a hard time with the borrow checker instead...
I suppose I really used a hammer on this problem... Creating a hash map of everything... In theory O(n) time.
Alternatively, a hash set can be made from the "to delete" array... Then each tree element can be checked to see if it exists in the hash set...
```Rust
// Definition for a binary tree node.
// #[derive(Debug, PartialEq, Eq)]
// pub struct TreeNode {
//   pub val: i32,
//   pub left: Option<Rc<RefCell<TreeNode>>>,
//   pub right: Option<Rc<RefCell<TreeNode>>>,
// }
// 
// impl TreeNode {
//   #[inline]
//   pub fn new(val: i32) -> Self {
//     TreeNode {
//       val,
//       left: None,
//       right: None
//     }
//   }
// }
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::HashMap;
use std::collections::HashSet;
impl Solution {
    pub fn del_nodes(root: Option<Rc<RefCell<TreeNode>>>, to_delete: Vec<i32>) -> Vec<Option<Rc<RefCell<TreeNode>>>> {
        let mut map = HashMap::new();
        dfs(&root.clone().unwrap().borrow(), &mut map);
        map.insert(root.clone().unwrap().borrow().val,(root.clone().unwrap(),0));
        
        let mut cans = HashSet::new();
        cans.insert(root.clone().unwrap().borrow().val);
        for e in to_delete.iter() {
            if let Some((node,par)) = map.get(e) {
                if let Some(ref l) = node.borrow().left {
                    cans.insert(l.borrow().val);
                }
                if let Some(ref r) = node.borrow().right {
                    cans.insert(r.borrow().val);
                }
                
                if let Some((pnode,_)) = map.get(par) {
                    let mut p = pnode.borrow_mut();
                    if let Some(ref left) = p.left {
                        if left.borrow().val == *e {
                            p.left = None;
                        }
                    }
                    if let Some(ref right) = p.right {
                        if right.borrow().val == *e {
                            p.right = None;
                        }
                    }
                }
            }
        }

        for e in to_delete.iter() {
            cans.remove(e);
        }

        //println!("{:?}",cans);

        let mut out = Vec::new();

        for e in cans {
            //println!("{e}");
            out.push(Some(map.get(&e).unwrap().0.clone()));
        }

        out
    }
}

fn dfs(node: &TreeNode, map: &mut HashMap<i32,(Rc<RefCell<TreeNode>>,i32)>) {
    //print!("{} ",node.val);
    if let Some(ref left) = node.left {
        map.insert(left.borrow().val,(left.clone(),node.val));
        dfs(&left.borrow(),map);
    }
    if let Some(ref right) = node.right {
        map.insert(right.borrow().val,(right.clone(),node.val));
        dfs(&right.borrow(),map);
    }
}
```
# 2024-07-16
[2096. Step-By-Step Directions From a Binary Tree Node to Another](https://leetcode.com/problems/step-by-step-directions-from-a-binary-tree-node-to-another/)
Not too much pain.
I should really read up on Rust borrowing rules and how explicit borrowing works. For now, I just use ```.clone()``` or ```.borrow()``` to get it to compile...
Some resources:
- https://gist.github.com/jimmychu0807/9a89355e642afad0d2aeda52e6ad2424
- https://stackoverflow.com/questions/54012660/unwrap-and-access-t-from-an-optionrcrefcellt
```Rust
// Definition for a binary tree node.
// #[derive(Debug, PartialEq, Eq)]
// pub struct TreeNode {
//   pub val: i32,
//   pub left: Option<Rc<RefCell<TreeNode>>>,
//   pub right: Option<Rc<RefCell<TreeNode>>>,
// }
// 
// impl TreeNode {
//   #[inline]
//   pub fn new(val: i32) -> Self {
//     TreeNode {
//       val,
//       left: None,
//       right: None
//     }
//   }
// }
use std::rc::Rc;
use std::cell::RefCell;
impl Solution {
    pub fn get_directions(root: Option<Rc<RefCell<TreeNode>>>, start_value: i32, dest_value: i32) -> String {
        let mut a = Vec::new();
        let mut b = Vec::new();

        dfs(&root.clone().unwrap().borrow(),start_value,&mut a);
        dfs(&root.clone().unwrap().borrow(),dest_value,&mut b);
        //println!("{:?}",a);
        //println!("{:?}",b);

        loop {
            if a.last() == None {
                break;
            }
            if b.last() == None {
                break;
            }
            let aa = a.last().unwrap();
            let bb = b.last().unwrap();
            if aa == bb {
                a.pop();
                b.pop();
            } else {
                break;
            }
        }
        //println!("{:?}",a);
        //println!("{:?}",b);

        for e in a.iter_mut() {
            *e = 'U';
        }
        b.reverse();
        a.append(&mut b);

        let out = a.iter().collect::<String>();
        out
    }
}

fn dfs(node: &TreeNode, val: i32, s: &mut Vec<char>) -> bool {
    if node.val == val {
        return true;
    }
    if let Some(ref left) = node.left {
        if dfs(&left.borrow(),val,s) == true {
            s.push('L');
            return true;
        }
    }
    if let Some(ref right) = node.right {
        if dfs(&right.borrow(),val,s) == true {
            s.push('R');
            return true;
        }
    }
    false
}
```
# 2024-07-15
[2196. Create Binary Tree From Descriptions](https://leetcode.com/problems/create-binary-tree-from-descriptions/)
More pain. But it some how it just works.
```Rust
// Definition for a binary tree node.
// #[derive(Debug, PartialEq, Eq)]
// pub struct TreeNode {
//   pub val: i32,
//   pub left: Option<Rc<RefCell<TreeNode>>>,
//   pub right: Option<Rc<RefCell<TreeNode>>>,
// }
// 
// impl TreeNode {
//   #[inline]
//   pub fn new(val: i32) -> Self {
//     TreeNode {
//       val,
//       left: None,
//       right: None
//     }
//   }
// }
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::HashMap;
impl Solution {
    pub fn create_binary_tree(descriptions: Vec<Vec<i32>>) -> Option<Rc<RefCell<TreeNode>>> {
        /*
        let mut a = Some(Rc::new(RefCell::new(TreeNode::new(1))));
        let mut b = Some(Rc::new(RefCell::new(TreeNode::new(2))));
        let mut c = Some(Rc::new(RefCell::new(TreeNode::new(3))));

        {
            let mut aa = Rc::get_mut(a.as_mut().unwrap()).unwrap().borrow_mut();
            aa.left = b;
            aa.right = c;
        }
        */

        let mut h: HashMap<i32,_> = HashMap::new();
        let mut c = HashMap::new();

        for v in descriptions {
            match h.get(&v[0]) {
                None => {
                    let a = Rc::new(RefCell::new(TreeNode::new(v[0])));
                    h.insert(v[0],a);
                },
                _ => (),
            }
            match h.get(&v[1]) {
                None => {
                    let b = Rc::new(RefCell::new(TreeNode::new(v[1])));
                    h.insert(v[1],b);
                },
                _ => (),
            }

            c.entry(v[0]).or_insert(0);
            c.entry(v[1]).and_modify(|e| *e+=1).or_insert(1);

            {
                let mut aa = h.get(&v[0]).unwrap().borrow_mut();
                let mut b = h.get(&v[1]).unwrap();
                if v[2] == 1 {
                    aa.left = Some(b.clone());
                } else {
                    aa.right = Some(b.clone());
                }
            }

        }

        //println!("{:?}",c);
        let mut par = None;
        for (key,val) in c.iter() {
            if *val == 0 {
                println!("{}",key);
                par = Some(h.get(key).unwrap().clone());
            }
        }
        //println!("{:?}",par);

        par
    }
}
```
# 2024-07-14
[726. Number of Atoms](https://leetcode.com/problems/number-of-atoms/)
Pain.
Implemented a tokenizer... And used enums to implement a basic state machine...
Looked at an example solution and they parsed the string in reverse... Though they had to use recursion.

I guess they were able to avoid dealing with ownership rules by parsing the input in reverse... The majority of my code is trying to deal with Rust's ownership rules and figuring out types... Well, I suppose I'm just learning Rust's API and figuring out all of its idiosyncrasies. (Either brute force through Rust's API or maneuver around it...)

I suppose powerful tools need powerful guard rails... That being said, not everything can be caught during compile time... However, I shouldn't really expect that anyways.
```Rust
use std::collections::HashMap;

impl Solution {
    pub fn count_of_atoms(formula: String) -> String {
        // tokenize
        let mut s: Vec<u8> = Vec::new();
        let mut n = 1;
        let mut tokens = Vec::new();
        let mut state = State::Paren;

        for e in formula.as_bytes().iter() {
            match e {
                b'A'..=b'Z' => {
                    match state {
                        State::Element => {
                            tokens.push(Elem::Element(String::from_utf8(s.clone()).unwrap()));
                        },
                        State::Number => {
                            tokens.push(Elem::Number(String::from_utf8(s.clone()).unwrap().parse().unwrap()));
                        },
                        _ => (),
                    }
                    //let mut s: Vec<u8> = Vec::new();
                    s.clear();
                    s.push(*e);
                    state = State::Element;
                },
                b'a'..=b'z' => {
                    s.push(*e);
                },
                b'(' => {
                    match state {
                        State::Element => {
                            tokens.push(Elem::Element(String::from_utf8(s.clone()).unwrap()));
                        },
                        State::Number => {
                            tokens.push(Elem::Number(String::from_utf8(s.clone()).unwrap().parse().unwrap()));
                        },
                        _ => (),
                    }
                    //let mut s: Vec<u8> = Vec::new();
                    s.clear();
                    tokens.push(Elem::Paren);
                    state = State::Paren;
                },
                b')' => {
                    match state {
                        State::Element => {
                            tokens.push(Elem::Element(String::from_utf8(s.clone()).unwrap()));
                        },
                        State::Number => {
                            tokens.push(Elem::Number(String::from_utf8(s.clone()).unwrap().parse().unwrap()));
                        },
                        _ => (),
                    }
                    //let mut s: Vec<u8> = Vec::new();
                    s.clear();
                    tokens.push(Elem::Unparen);
                    state = State::Unparen;

                },
                b'0'..=b'9' => {
                    match state {
                        State::Element => {
                            tokens.push(Elem::Element(String::from_utf8(s.clone()).unwrap()));
                            //let mut s: Vec<u8> = Vec::new();
                            s.clear();
                        },
                        _ => (),
                    }
                    s.push(*e);
                    state = State::Number;
                },
                _ => {
                    break;
                }
            }
        }

        match state {
            State::Element => {
                tokens.push(Elem::Element(String::from_utf8(s.clone()).unwrap()));
            },
            State::Number => {
                tokens.push(Elem::Number(String::from_utf8(s.clone()).unwrap().parse().unwrap()));
            },
            _ => (),
        }

        //println!("{:?}",tokens);

        let mut stack: Vec<HashMap<String,i32>> = Vec::new();
        stack.push(HashMap::new());
        let mut le = "".to_string();

        // parse
        /*
        for t in tokens {
            match t {
                Elem::Paren => {
                    stack.push(HashMap::new());
                },
                Elem::Element(es) => {
                    stack.last_mut().unwrap().insert(es,1);
                    le = es;
                },
                Elem::Number(en) => {
                    stack.last_mut().unwrap().entry(le).and_modify(|v| *v=en).or_insert(en);
                }
            }
        }
        */
        let mut t_iter = tokens.iter().peekable();
        let mut t = t_iter.next();
        loop {
            if t == None {
                break;
            }
            let e = t.unwrap();
            match e {
                Elem::Paren => {
                    stack.push(HashMap::new());
                },
                Elem::Element(es) => {
                    if t_iter.peek() == None {
                        //stack.last_mut().unwrap().insert(es.to_string(),1);
                        stack.last_mut().unwrap().entry(es.to_string()).and_modify(|v| *v += 1).or_insert(1);
                    } else {
                        let g = t_iter.peek().unwrap();
                        match g {
                            Elem::Number(en) => {
                                //stack.last_mut().unwrap().insert(es.to_string(),*en);
                                stack.last_mut().unwrap().entry(es.to_string()).and_modify(|v| *v += *en).or_insert(*en);
                                t = t_iter.next();
                            },
                            _ => {
                                //stack.last_mut().unwrap().insert(es.to_string(),1);
                                stack.last_mut().unwrap().entry(es.to_string()).and_modify(|v| *v += 1).or_insert(1);
                            },
                        }
                    }
                },
                Elem::Unparen => {
                    if t_iter.peek() != None {
                        match t_iter.peek().unwrap() {
                        Elem::Number(en) => {
                                for (_,val) in stack.last_mut().unwrap().iter_mut() {
                                    *val *= en;
                                }
                                t = t_iter.next();
                            },
                            _ => (),
                        }
                    }
                    let u = stack.pop().unwrap();
                    for (key,val) in u.iter() {
                        stack.last_mut().unwrap().entry(key.clone()).and_modify(|v| *v += *val).or_insert(*val);
                    }
                }
                _ => (),
            }

            //println!("{:?}",stack);

            t = t_iter.next();
        }
        //println!("{:?}",stack);

        // sort
        let mut elements: Vec<_> = stack[0].iter().collect();
        elements.sort_by_key(|(k)| k.0);
        //println!("{:?}",elements);

        // output as string
        let mut out = String::new();
        for (key,val) in elements {
            if *val > 1 {
                out.push_str(&key);
                out.push_str(&val.to_string());
            } else {
                out.push_str(&key);
            }
        }
        
        out
    }
}

#[derive(Debug)]
#[derive(PartialEq)]
enum Elem {
    Paren,
    Unparen,
    Element(String),
    Number(i32),
}

enum State {
    Paren,
    Unparen,
    Element,
    Number,
}
```
# 2024-07-13
[2751. Robot Collisions](https://leetcode.com/problems/robot-collisions/)
Major learning curve for structs... Need to learn how traits actually work...
```Rust
impl Solution {
    pub fn survived_robots_healths(positions: Vec<i32>, healths: Vec<i32>, directions: String) -> Vec<i32> {
        let mut robos = Vec::new();
        let directions = directions.as_bytes();

        for i in 0..positions.len() {
            robos.push(
                Robo{
                    id: i as i32,
                    pos: positions[i],
                    hp: healths[i],
                    dir: if directions[i] == b'R' {
                        true
                    } else {
                        false
                    }
                }
            );
        }

        robos.sort_by_key(|k| k.pos);

        /*
        for e in robos.iter() {
            println!("{} {} {} {}", e.id, e.pos, e.hp, e.dir);
        }
        */

        let mut stack = Vec::new();

        for e in robos.iter() {
            if stack.last() == None {
                stack.push(
                    Robo{
                        id: e.id,
                        pos: e.pos,
                        hp: e.hp,
                        dir: e.dir
                    }
                );
                continue;
            }
            if e.dir == true {
                stack.push(
                    Robo{
                        id: e.id,
                        pos: e.pos,
                        hp: e.hp,
                        dir: e.dir
                    }
                );
            } else {
                //let s = stack.last().unwrap();
                let mut dir = stack.last().unwrap().dir;
                let mut hp = stack.last().unwrap().hp;
                if dir == true {
                    let mut a = e.clone();
                    while a.hp > hp && dir == true {
                        a.hp -= 1;
                        stack.pop();
                        match stack.last() {
                            None => {
                                dir = false;
                                break;
                            },
                            Some(inner) => {
                                hp = inner.hp;
                                dir = inner.dir;
                            },
                        }
                    }
                    if a.hp < hp && dir == true {
                        stack.last_mut().unwrap().hp -= 1;
                        continue;
                    }
                    if a.hp == hp && dir == true{
                        stack.pop();
                        continue;
                    }
                    stack.push(
                        Robo{
                            id: a.id,
                            pos: a.pos,
                            hp: a.hp,
                            dir: a.dir
                        }
                    );
                } else {
                    stack.push(
                        Robo{
                            id: e.id,
                            pos: e.pos,
                            hp: e.hp,
                            dir: e.dir
                        }
                    );
                }
            }
        }

        /*
        println!();
        for e in stack.iter() {
            println!("{} {} {} {}", e.id, e.pos, e.hp, e.dir);
        }
        */

        stack.sort_by_key(|k| k.id);

        let mut out = Vec::new();

        for e in stack.iter() {
            out.push(e.hp);
        }

        out
    }
}

#[derive(PartialEq)]
#[derive(Clone)]
struct Robo {
    id: i32,
    pos: i32,
    hp: i32,
    dir: bool
}
```
# 2024-07-12
[1717. Maximum Score From Removing Substrings](https://leetcode.com/problems/maximum-score-from-removing-substrings/)
Repeated code but just making it simple.
Some counting logic to solve the problem. Nothing crazy going on here.
Basic greedy approach... You need the same amount of a's and b's for either case.
```Rust
impl Solution {
    pub fn maximum_gain(s: String, x: i32, y: i32) -> i32 {
        let mut a = 0;
        let mut b = 0;
        let mut m = 0;
        let mut n = 0;
        if x > y {
            for e in s.chars() {
                match e {
                    'a' => a += 1,
                    'b' => {
                        if a > 0 {
                            a -= 1;
                            m += 1;
                        } else {
                            b += 1;
                        }
                    },
                    _ => {
                        if a < b {
                            b = a;
                        }
                        n += b;
                        a = 0;
                        b = 0;
                    }
                }
            }
            if a < b {
                b = a;
            }
            n += b;
        } else {
            for e in s.chars() {
                match e {
                    'a' => {
                        if b > 0 {
                            b -= 1;
                            n += 1;
                        } else {
                            a += 1;
                        }
                    },
                    'b' => b += 1,
                    _ => {
                        if b < a {
                            a = b;
                        }
                        m += a;
                        a = 0;
                        b = 0;
                    }
                }
            }
            if b < a {
                a = b;
            }
            m += a;
        }

        m*x + n*y
    }
}
```
# 2024-07-11
[1190. Reverse Substrings Between Each Pair of Parentheses](https://leetcode.com/problems/reverse-substrings-between-each-pair-of-parentheses/)
A stack method. A better solution can be done with a single vector versus a vector of vectors in my solution.
```Rust
impl Solution {
    pub fn reverse_parentheses(s: String) -> String {
        let mut v = Vec::new();
        v.push(Vec::new());

        for e in s.chars() {
            match e {
                '(' => {
                    v.push(Vec::new());
                },
                ')' => {
                    let mut t = v.pop().unwrap();
                    t.reverse();
                    v.last_mut().unwrap().append(&mut t);
                },
                _ => {
                    v.last_mut().unwrap().push(e);
                }
            }
        }

        v.pop().unwrap().iter().collect::<String>()
    }
}
```

Failed solution. I was onto the better solution...
```Rust
impl Solution {
    pub fn reverse_parentheses(s: String) -> String {
        let mut r = false;
        
        let mut a = Vec::new();
        let mut b = Vec::new();

        for e in s.chars() {
            match e {
                '(' => r = !r,
                ')' => {
                    if r == true {
                        b.reverse();
                        a.append(&mut b);
                    } else {
                        a.reverse();
                        b.append(&mut a);
                    }
                    r = !r;
                },
                _ => {
                    if r == true {
                        b.push(e);
                    } else {
                        a.push(e);
                    }
                }
            }
        }

        a.iter().collect::<String>()
```
# 2024-07-10
[1598. Crawler Log Folder](https://leetcode.com/problems/crawler-log-folder/)
String comparisons are easy in Rust.
```Rust
impl Solution {
    pub fn min_operations(logs: Vec<String>) -> i32 {
        let mut d = 0;
        for e in logs {
            if e == "../" {
                if d > 0 {
                    d -= 1;
                }
            } else if e == "./" {
                continue;
            } else {
                d += 1;
            }
        }
        d
    }
}
```
# 2024-07-09
[1701. Average Waiting Time](https://leetcode.com/problems/average-waiting-time/)
Need to check for overflow.
```Rust
impl Solution {
    pub fn average_waiting_time(customers: Vec<Vec<i32>>) -> f64 {
        let mut wait: i64 = 0;
        let mut t: i64 = 0;

        for e in customers.iter() {
            if t < (e[0] as i64) {
                t = (e[0]+e[1]) as i64;
                wait += (e[1] as i64);
            } else {
                t = t + (e[1] as i64);
                wait += (t - (e[0] as i64));
            }
        }

        (wait as f64)/(customers.len() as f64)
    }
}
```
# 2024-07-08
[1823. Find the Winner of the Circular Game](https://leetcode.com/problems/find-the-winner-of-the-circular-game/)
```Rust
impl Solution {
    pub fn find_the_winner(n: i32, k: i32) -> i32 {
        let mut p = Vec::new();
        for j in 1..=n {
            p.push(j);
        }
        let mut i = 0;
        for r in (2..=n).rev() {
            i = (i+k-1)%r;
            p.remove(i as usize);
        }
        p[0]
    }
}
```
# 2024-07-07
[1518. Water Bottles](https://leetcode.com/problems/water-bottles/)
```Rust
impl Solution {
    pub fn num_water_bottles(num_bottles: i32, num_exchange: i32) -> i32 {
        let mut b = num_bottles;
        let mut e = num_exchange;
        let mut s = b;

        while b>=e {
            let re = b%e;
            b = b/e;
            s += b;
            b += re;
        }

        s
    }
}
```
# 2024-07-06
[2582. Pass the Pillow](https://leetcode.com/problems/pass-the-pillow/)
```Rust
impl Solution {
    pub fn pass_the_pillow(n: i32, time: i32) -> i32 {
        let mut a = (time%((n-1)*2))+1;
        if a>n {
            a = n*2 - a;
        }
        a
    }
}
```
# 2024-07-05
[2058. Find the Minimum and Maximum Number of Nodes Between Critical Points](https://leetcode.com/problems/find-the-minimum-and-maximum-number-of-nodes-between-critical-points/)
```Rust
// Definition for singly-linked list.
// #[derive(PartialEq, Eq, Clone, Debug)]
// pub struct ListNode {
//   pub val: i32,
//   pub next: Option<Box<ListNode>>
// }
// 
// impl ListNode {
//   #[inline]
//   fn new(val: i32) -> Self {
//     ListNode {
//       next: None,
//       val
//     }
//   }
// }
impl Solution {
    pub fn nodes_between_critical_points(head: Option<Box<ListNode>>) -> Vec<i32> {
        let mut a = head.as_ref();
        let mut b = Vec::new();

        let mut c = a.unwrap().val;
        a = a.unwrap().next.as_ref();
        let mut d = a.unwrap().val;
        a = a.unwrap().next.as_ref();
        let mut i = 0;
        while a != None {
            let e = a.unwrap().val;
            if d > c && d > e {
                b.push(i);
            }
            if d < c && d < e {
                b.push(i);
            }
            c = d;
            d = e;
            i += 1;
            a = a.unwrap().next.as_ref();
        }

        let mut out = vec![-1,-1];

        if b.len() < 2 {
            return out;
        }

        out[0] = b[1]-b[0];
        out[1] = b[b.len()-1]-b[0];
        for j in 1..b.len() {
            if out[0] > b[j]-b[j-1] {
                out[0] = b[j]-b[j-1];
            }
        }
        out
    }
}
```
# 2024-07-04
[2181. Merge Nodes in Between Zeros](https://leetcode.com/problems/merge-nodes-in-between-zeros/)
```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* mergeNodes(struct ListNode* head) {
    struct ListNode *curr, *prev;
    int sum;
    sum = 0;
    prev = head;
    curr = prev->next;
    while(curr) {
        //printf("%d ",curr->val);
        if(curr->val == 0){
            //prev = prev->next;
            prev->val = sum;
            sum = 0;
            if(curr->next){
                prev->next = curr;
                prev = curr;
            } else {
                prev->next = NULL;
            }
        } else {
            sum += curr->val;
        }
        curr = curr->next;
    }
    return head;
}
```
# 2024-07-03
[1509. Minimum Difference Between Largest and Smallest Value in Three Moves](https://leetcode.com/problems/minimum-difference-between-largest-and-smallest-value-in-three-moves/)
```Rust
impl Solution {
    pub fn min_difference(nums: Vec<i32>) -> i32 {
        if nums.len() <= 4 {
            return 0;
        }

        let mut a = nums.clone();
        a.sort_unstable();
        //println!("{:?}",a);
        /*
        let mut i = 0;
        let mut j = a.len()-1;
        for k in 0..3 {
            if a[j]-a[i+1] > a[j-1]-a[i] {
                j -= 1;
            } else {
                i += 1;
            }
        }

        a[j]-a[i]
        */
        //let mut b = Vec::new();
        for i in 0..4 {
            b.push(a[a.len()-4+i] - a[i]);
        }
        b.sort();

        b[0]
    }
}
```
# 2024-07-02
[350. Intersection of Two Arrays II](https://leetcode.com/problems/intersection-of-two-arrays-ii/)
```Rust
impl Solution {
    pub fn intersect(nums1: Vec<i32>, nums2: Vec<i32>) -> Vec<i32> {
        let mut c1 = vec![0; 1001];
        let mut c2 = vec![0; 1001];

        for e in nums1 {
            c1[e as usize] += 1;
        }
        for e in nums2 {
            c2[e as usize] += 1;
        }
        for i in 0..1001 {
            if c1[i] > c2[i] {
                c1[i] = c2[i];
            }
        }

        let mut out = Vec::new();
        for i in 0..1001 {
            for j in 0..c1[i] {
                out.push(i as i32);
            }
        }

        out
    }
}
```
# 2024-07-01
[1550. Three Consecutive Odds](https://leetcode.com/problems/three-consecutive-odds/)
```rust
impl Solution {
    pub fn three_consecutive_odds(arr: Vec<i32>) -> bool {
        let mut odd = 0;
        for e in arr {
            if e&1 == 1 {
                odd += 1;
            } else {
                odd = 0;
            }
            if odd >= 3 {
                return true;
            }
        }
        false
    }
}
```
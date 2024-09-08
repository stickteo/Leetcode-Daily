# 2024-08-31
[1514. Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/)

Rust requires complete "Ord" for a Binary Heap. Float 64 in Rust only implements Partial Ord. My solution? Convert it into a string and back... It's severely slow though.

Though not particularly slower... Apparently running the top "4ms" solution gives a 22ms solution in comparison to my 31ms solution. This means my solution takes around 1.5~3 times as long. Which is surprisingly quick...

Strangely it uses slightly less memory...

## Less Lazy
```Rust
impl Solution {
    pub fn max_probability(n: i32, edges: Vec<Vec<i32>>, succ_prob: Vec<f64>, start_node: i32, end_node: i32) -> f64 {
        let mut neighbors = vec![Vec::new(); n as usize];
        for (i,e) in edges.iter().enumerate() {
            neighbors[e[0] as usize].push((e[1] as usize,succ_prob[i]));
            neighbors[e[1] as usize].push((e[0] as usize,succ_prob[i]));
        }

        let mut prob = vec![0.0; n as usize];
        prob[start_node as usize] = 1.0;
        dijkstra(&neighbors, &mut prob, start_node as usize, end_node as usize);

        prob[end_node as usize]
    }
}

use std::collections::BinaryHeap;
fn dijkstra(neighbors: &Vec<Vec<(usize,f64)>>, prob: &mut Vec<f64>, start: usize, end: usize) {
    let mut heap = BinaryHeap::new();
    heap.push((format!("{:0.6}",prob[start]),start));

    while !heap.is_empty() {
        let (s,n) = heap.pop().unwrap();
        if n==end {
            return;
        }
        let p: f64 = s.parse().unwrap();

        for &(edge,pp) in neighbors[n].iter() {
            let q = p*pp;
            if q > prob[edge] {
                prob[edge] = q;
                heap.push((format!("{:0.6}",q),edge));
            }
        }
    }
}
```

## Lazy
```Rust
impl Solution {
    pub fn max_probability(n: i32, edges: Vec<Vec<i32>>, succ_prob: Vec<f64>, start_node: i32, end_node: i32) -> f64 {
        let mut neighbors = vec![Vec::new(); n as usize];
        for (i,e) in edges.iter().enumerate() {
            neighbors[e[0] as usize].push((e[1] as usize,succ_prob[i]));
            neighbors[e[1] as usize].push((e[0] as usize,succ_prob[i]));
        }
        //println!("{:?}",neighbors);

        let mut prob = vec![0.0; n as usize];
        prob[start_node as usize] = 1.0;
        dijkstra(&neighbors, &mut prob, start_node as usize);
        //println!("{:?}",prob);

        prob[end_node as usize]
    }
}

use std::collections::BinaryHeap;
fn dijkstra(neighbors: &Vec<Vec<(usize,f64)>>, prob: &mut Vec<f64>, start: usize) {
    let mut heap = BinaryHeap::new();
    heap.push((format!("{:0.6}",prob[start]),start));

    while !heap.is_empty() {
        let (s,n) = heap.pop().unwrap();
        //print!("{} ",s);
        let p: f64 = s.parse().unwrap();

        for &(edge,pp) in neighbors[n].iter() {
            let q = p*pp;
            if q > prob[edge] {
                prob[edge] = q;
                heap.push((format!("{:0.6}",q),edge));
            }
        }
    }
}
```
# 2024-08-30
[2699. Modify Graph Edge Weights](https://leetcode.com/problems/modify-graph-edge-weights/)

I need to do a different approach.

Tests:
```
3
[[0,1,-1],[0,2,5]]
0
2
6
4
[[1,0,4],[1,2,3],[2,3,5],[0,3,-1]]
0
2
6
4
[[0,1,-1],[0,2,-1],[1,3,-1],[2,3,-1]]
0
3
8
4
[[0,1,1],[0,2,-1],[1,3,1],[2,3,-1]]
0
3
8
3
[[0,1,10],[1,2,-1]]
0
2
5
3
[[0,1,10],[1,2,1]]
0
2
11
4
[[3,0,-1],[1,2,-1],[2,3,-1],[1,3,9],[2,0,5]]
0
1
7
5
[[0,2,5],[2,1,-1],[2,4,3],[3,4,5],[4,0,1],[0,3,-1],[2,3,-1]]
0
1
9
```

## Attempt
```Rust
use std::collections::BinaryHeap;

impl Solution {
    pub fn modified_graph_edges(n: i32, edges: Vec<Vec<i32>>, source: i32, destination: i32, target: i32) -> Vec<Vec<i32>> {
        let mut neighbors = vec![Vec::new(); n as usize];
        let mut modable = vec![false; edges.len()];
        let mut edges = edges;

        for (i,e) in edges.iter_mut().enumerate() {
            if e[2] == -1 {
                modable[i] = true;
                // set weight to minimum of 1
                e[2] = 1;
            }
            neighbors[e[0] as usize].push((e[1],e[2],i as i32));
            neighbors[e[1] as usize].push((e[0],e[2],i as i32));
        }

        let mut dist = vec![i32::MAX; n as usize];
        let par = dijkstra(&neighbors, &mut dist, source, destination);
        
        /*
        println!("{:?}",par);
        println!("{:?}",dist);
        */

        // shortest possible distance is greater than target
        if dist[destination as usize] > target {
            return Vec::new();
        }

        // shortest distance equal to target
        if dist[destination as usize] == target {
            return edges;
        }

        // shortest path
        let mut shortest = vec![false; edges.len()];
        let mut u = par[destination as usize];
        while u.0 != par[u.0 as usize].0 {
            //print!("{} ",u.0);
            shortest[u.1 as usize] = true;
            u = par[u.0 as usize];
        }
        //println!("{} ",u.0);
        shortest[u.1 as usize] = true;

        let a: Vec<bool> = modable.iter().zip(shortest.iter()).map(|(&x,&y)| x && y).collect();
        let b: Vec<bool> = modable.iter().zip(shortest.iter()).map(|(&x,&y)| x && !y).collect();
        /*
        println!("{:?}",modable);
        println!("{:?}",shortest);
        println!("{:?}",a);
        println!("{:?}",b);
        */

        // no modifiable edges
        let count = a.iter().filter(|&&x| x).count();
        if count == 0 {
            return Vec::new();
        }

        let mut left = target - dist[destination as usize];
        let c = left/(count as i32);
        for i in 0..edges.len() {
            if a[i] {
                //edges[i][2] = 1 + target - dist[destination as usize];
                edges[i][2] += c;
            }
        }
        let mut d = left%(count as i32);
        for i in 0..edges.len() {
            if d == 0 {
                break;
            }
            if a[i] {
                //edges[i][2] = 1 + target - dist[destination as usize];
                edges[i][2] += 1;
                d -= 1;
            }
        }
        for i in 0..edges.len() {
            if b[i] {
                edges[i][2] = 10_000_000;
            }
        }

        // check if shortest path is modifiable
        let mut dist2 = vec![i32::MAX; n as usize];
        let mut neighbors2 = vec![Vec::new(); n as usize];
        for (i,e) in edges.iter_mut().enumerate() {
            if modable[i] {
                // infinite edge weight
                continue;
                //e[2] = 10_000_000;
            }
            neighbors2[e[0] as usize].push((e[1],e[2],i as i32));
            neighbors2[e[1] as usize].push((e[0],e[2],i as i32));
        }
        dijkstra(&neighbors2, &mut dist2, source, destination);
        //println!("{:?}",dist2);
        if dist2[destination as usize] < target {
            //println!("a");
            return Vec::new();
        }
        
        edges
    }
}

fn dijkstra(neighbors: &Vec<Vec<(i32,i32,i32)>>, dist: &mut Vec<i32>, start: i32, end: i32) -> Vec<(i32,i32)> {
    let mut heap = BinaryHeap::new();
    let mut par = vec![(0,0); dist.len()];
    par[start as usize] = (start, -1);
    dist[start as usize] = 0;
    heap.push((0,start));

    //println!("{:?}",neighbors);

    while !heap.is_empty() {
        let (d,next) = heap.pop().unwrap();
        for &(node,cost,edge) in neighbors[next as usize].iter() {
            let dd = d + cost;
            if dd < dist[node as usize] {
                dist[node as usize] = dd;
                par[node as usize] = (next, edge);
                heap.push((dd,node));
            }
        }
    }

    par
}
```
# 2024-08-29
[947. Most Stones Removed with Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/)

Heh, got it first try. Though it did help this question is based on using Union-Find.

The key insight is that stones can be collected into groups. Then for each group, you can remove length-1 stones.

Rust matching is really dope man. It looks so freaking clean.

```Rust
impl Solution {
    pub fn remove_stones(stones: Vec<Vec<i32>>) -> i32 {
        let mut x = vec![0; 10001];
        let mut y = vec![0; 10001];
        let mut par = vec![0,1];
        let mut count = vec![0,0];

        let mut g = 1;
        for s in stones.iter() {
            let a = s[0] as usize;
            let b = s[1] as usize;
            match (x[a],y[b]) {
                (0,0) => {
                    x[a] = g;
                    y[b] = g;
                    count[g] += 1;
                    g += 1;
                    count.push(0);
                    par.push(g);
                }
                (0,d) => {
                    x[a] = d;
                    count[d] += 1;
                }
                (c,0) => {
                    y[b] = c;
                    count[c] += 1;
                }
                (c,d) => {
                    let e = merge(&mut par,c,d);
                    count[e] += 1;
                }
            }
        }

        /*
        println!("p: {:?}",par);
        println!("c: {:?}",count);
        */

        for i in 1..count.len() {
            if count[i]>0 && par[i]!=i {
                let a = find(&par,i);
                count[a] += count[i];
                count[i] = 0;
            }
        }
        count.iter().filter(|&&x| x>1).map(|x| x-1).sum()
    }
}

fn find(par: & Vec<usize>, a: usize) -> usize {
    let mut a = a;
    while a != par[a as usize] {
        a = par[a as usize];
    }
    a
}

fn merge(par: &mut Vec<usize>, a: usize, b: usize) -> usize {
    let mut a = find(par,a);
    let mut b = find(par,b);

    if a == b {
        return a;
    }

    if a < b {
        par[b as usize] = a;
        return a;
    } else {
        par[a as usize] = b;
        return b;
    }
}
```

# 2024-08-28
[1905. Count Sub Islands](https://leetcode.com/problems/count-sub-islands/)

```Rust
impl Solution {
    pub fn count_sub_islands(grid1: Vec<Vec<i32>>, grid2: Vec<Vec<i32>>) -> i32 {
        let mut g1 = vec![vec![0; grid1[0].len()]; grid1.len()];
        let mut g2 = vec![vec![0; grid2[0].len()]; grid2.len()];
        let mut p1 = Vec::new();
        p1.push(0);
        let mut p2 = Vec::new();
        p2.push(0);

        process(&grid1, &mut g1, &mut p1);
        process(&grid2, &mut g2, &mut p2);

        /*
        print_grid(&g1);
        println!();
        print_grid(&g2);
        */
        

        let mut sub = vec![-1; p2.len()];
        for i in 0..g2.len() {
            for j in 0..g2[0].len() {
                if g2[i][j] == 0 {
                    continue;
                }
                match sub[g2[i][j] as usize] {
                    -1 => {
                        sub[g2[i][j] as usize] = g1[i][j];
                    }
                    0 => {
                        continue;
                    }
                    x => {
                        if x != g1[i][j] {
                            sub[g2[i][j] as usize] = 0;
                        }
                    }
                }
            }
        }
        //println!("{:?}",sub);

        sub.iter().filter(|&&x| x>0).count() as i32
    }
}

fn process(grid: &Vec<Vec<i32>>, g: &mut Vec<Vec<i32>>, p: &mut Vec<i32>) {
    let mut c = 1;
    for i in 0..grid.len() {
        for j in 0..grid[0].len() {
            if grid[i][j] == 1 {
                let l = if i>0 {
                    g[i-1][j]
                } else {
                    0
                };
                let u = if j>0 {
                    g[i][j-1]
                } else {
                    0
                };
                match (l,u) {
                    (0,0) => {
                        g[i][j] = c;
                        p.push(c);
                        c += 1;
                    }
                    (0,u) => {
                        g[i][j] = u;
                    }
                    (l,0) => {
                        g[i][j] = l;
                    }
                    (l,u) => {
                        g[i][j] = merge(p,l,u);
                    }
                }
            }
        }
    }

    for e in g.iter_mut() {
        for f in e.iter_mut() {
            *f = find(&p, *f);
        }
    }
}

fn find(p: &Vec<i32>, n: i32) -> i32 {
    let mut n = n;
    while n != p[n as usize] {
        n = p[n as usize];
    }
    n
}

fn merge(p: &mut Vec<i32>, a: i32, b: i32) -> i32 {
    let a = find(&p, a);
    let b = find(&p, b);

    if a == b {
        return a;
    }

    if a < b {
        p[b as usize] = a;
        return a;
    } else {
        p[a as usize] = b;
        return b;
    };
}

fn print_grid(g: &Vec<Vec<i32>>) {
    for e in g {
        for f in e {
            if *f > 0 {
                print!(" {:2}", f);
            } else {
                print!("   ");
            }
        }
        println!();
    }
}
```

# 2024-08-26
[590. N-ary Tree Postorder Traversal](https://leetcode.com/problems/n-ary-tree-postorder-traversal/)

```C
/**
 * Definition for a Node.
 * struct Node {
 *     int val;
 *     int numChildren;
 *     struct Node** children;
 * };
 */

/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

int* postorder(struct Node* root, int* returnSize) {
    if (root == NULL) {
        *returnSize = 0;
        return NULL;
    }

    int* out = malloc(sizeof(int)*10000);
    //struct Node** levels = malloc(sizeof(struct Node*)*1000);
    //int* cs = calloc(sizeof(int),1000);
    struct Node* levels[1000];
    int cs[1000];
    memset(cs,0,4000);
    int l = 0;
    int n = 0;
    levels[0] = root;

    while (l>=0) {
        if (cs[l] < levels[l]->numChildren) {
            levels[l+1] = levels[l]->children[cs[l]];
            cs[l]++;
            l++;
        } else {
            out[n] = levels[l]->val;
            n++;
            cs[l] = 0;
            l--;
        }
    }

    *returnSize = n;
    return out;
}
```

# 2024-08-25
[590. N-ary Tree Postorder Traversal](https://leetcode.com/problems/n-ary-tree-postorder-traversal/)

No Rust template so now back to good old C.

The iterative solution is faster. Allocating memory on the stack is faster than using malloc().

## Iterative
```C
int* postorder(struct Node* root, int* returnSize) {
    if (root == NULL) {
        *returnSize = 0;
        return NULL;
    }

    int* out = malloc(sizeof(int)*10000);
    //struct Node** levels = malloc(sizeof(struct Node*)*1000);
    //int* cs = calloc(sizeof(int),1000);
    struct Node* levels[1000];
    int cs[1000];
    memset(cs,0,4000);
    int l = 0;
    int n = 0;
    levels[0] = root;

    while (l>=0) {
        if (cs[l] < levels[l]->numChildren) {
            levels[l+1] = levels[l]->children[cs[l]];
            cs[l]++;
            l++;
        } else {
            out[n] = levels[l]->val;
            n++;
            cs[l] = 0;
            l--;
        }
    }

    *returnSize = n;
    return out;
}
```

## No Size Check
```C
int dfs(struct Node* node, int* out) {
    int n = 0;

    for (int i=0; i<node->numChildren; i++) {
        n += dfs(node->children[i], out+n);
    }

    out[n] = node->val;
    n += 1;

    return n;
}

int* postorder(struct Node* root, int* returnSize) {
    if (root == NULL) {
        *returnSize = 0;
        return NULL;
    }

    int* out = malloc(sizeof(int)*10000);

    *returnSize = dfs(root,out);
    return out;
}
```

## Typical Solution
```C
/**
 * Definition for a Node.
 * struct Node {
 *     int val;
 *     int numChildren;
 *     struct Node** children;
 * };
 */

/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
int dfs_size(struct Node* node) {
    int n = 1;

    for (int i=0; i<node->numChildren; i++) {
        n += dfs_size(node->children[i]);
    }

    return n;
}

int dfs(struct Node* node, int* out) {
    int n = 0;

    for (int i=0; i<node->numChildren; i++) {
        n += dfs(node->children[i], out+n);
    }

    out[n] = node->val;
    n += 1;

    return n;
}

int* postorder(struct Node* root, int* returnSize) {
    if (root == NULL) {
        *returnSize = 0;
        return NULL;
    }
    int n = dfs_size(root);
    //printf("%d\n", n);

    int* out = malloc(sizeof(int)*n);

    dfs(root,out);

    *returnSize = n;
    return out;
}
```
# 2024-08-24
[564. Find the Closest Palindrome](https://leetcode.com/problems/find-the-closest-palindrome/)

Not too hard. Only a few edge cases. Using Rust eliminates a lot of "boilerplate" stuff. String manipulation is relatively easy.

```Rust
impl Solution {
    pub fn nearest_palindromic(n: String) -> String {
        if n.len()&1 == 1 {
            let a: i64 = (&n[0..n.len()/2+1]).parse().unwrap();
            let ls = format!("{}",a-1);
            let ms = format!("{}",a);
            let hs = format!("{}",a+1);
            
            let l: i64 = if ls.len()<ms.len() {
                ls.chars().chain(ls.chars().rev())
                    .collect::<String>().parse().unwrap()
            } else {
                ls.chars().chain(ls.chars().take(n.len()/2)
                    .collect::<String>().chars().rev())
                    .collect::<String>().parse().unwrap()
            };

            let m: i64 = ms.chars().chain(ms.chars().take(n.len()/2)
                .collect::<String>().chars().rev())
                .collect::<String>().parse().unwrap();

            let h: i64 = hs.chars().chain(hs.chars().take(n.len()/2)
                .collect::<String>().chars().rev())
                .collect::<String>().parse().unwrap();

            let v: i64 = n.parse().unwrap();

            let mut a = Vec::new();
            if l<v {
                a.push((v-l,l));
            }
            if m<v {
                a.push((v-m,m));
            }
            if m>v {
                a.push((m-v,m));
            }
            if h>v {
                a.push((h-v,h));
            }
            a.sort();
            //println!("{:?}",a);

            return format!("{}",a[0].1);
        } else {
            let a: i64 = (&n[0..n.len()/2]).parse().unwrap();
            let ls = format!("{}",a-1);
            let ms = format!("{}",a);
            let hs = format!("{}",a+1);
            
            let l: i64 = if ls.len()<ms.len() {
                ls.chars().chain("9".chars()).chain(ls.chars().rev())
                    .collect::<String>().parse().unwrap()
            } else if a==1 {
                9
            } else {
                ls.chars().chain(ls.chars().take(n.len()/2)
                    .collect::<String>().chars().rev())
                    .collect::<String>().parse().unwrap()
            };

            let m: i64 = ms.chars().chain(ms.chars().take(n.len()/2)
                .collect::<String>().chars().rev())
                .collect::<String>().parse().unwrap();
            
            let h: i64 = hs.chars().chain(hs.chars().take(n.len()/2)
                .collect::<String>().chars().rev())
                .collect::<String>().parse().unwrap();

            let v: i64 = n.parse().unwrap();

            let mut a = Vec::new();
            if l<v {
                a.push((v-l,l));
            }
            if m<v {
                a.push((v-m,m));
            }
            if m>v {
                a.push((m-v,m));
            }
            if h>v {
                a.push((h-v,h));
            }
            a.sort();
            //println!("{:?}",a);

            return format!("{}",a[0].1);
        }
        n
    }
}
```

# 2024-08-23
[592. Fraction Addition and Subtraction](https://leetcode.com/problems/fraction-addition-and-subtraction/)

```Rust
impl Solution {
    pub fn fraction_addition(expression: String) -> String {
        // parse
        let v: Vec<&str> = expression.split(&['-','/','+']).collect();
        println!("{:?}",v);
        let u: Vec<&str> = expression.matches(&['-','/','+']).collect();
        println!("{:?}",u);

        let mut j = 0;
        let mut num = match v[j].parse::<i32>() {
            Ok(x) => x,
            Err(_) => 0
        };
        let mut den = 1;
        j += 1;

        let mut fracs = Vec::new();
        for f in u.iter() {
            match *f {
                "+" => {
                    fracs.push((num,den));
                    num = match v[j].parse::<i32>() {
                        Ok(x) => x,
                        Err(_) => 0
                    };
                    den = 1;
                    j += 1;
                }
                "-" => {
                    fracs.push((num,den));
                    num = match v[j].parse::<i32>() {
                        Ok(x) => -x,
                        Err(_) => 0
                    };
                    den = 1;
                    j += 1;
                }
                "/" => {
                    den = match v[j].parse::<i32>() {
                        Ok(x) => x,
                        Err(_) => 0
                    };
                    j += 1;
                }
                &_ => ()
            }
        }
        fracs.push((num,den));
        println!("{:?}",fracs);

        // calculate
        let mut sum = fracs[0];
        for f in fracs[1..].iter() {
            let mut g = gcd(sum.1,f.1);
            let mut n = f.1/g*sum.0 + sum.1/g*f.0;
            let mut d = sum.1/g*f.1;
            if n == 0 {
                d = 1;
            } else {
                if n < 0 {
                    g = gcd(-n,d);
                } else {
                    g = gcd(n,d);
                }
                n = n/g;
                d = d/g;
            }
            sum = (n,d);
        }

        //expression
        format!("{}/{}",sum.0,sum.1)
    }
}

fn gcd(a: i32, b: i32) -> i32 {
    let mut a = a;
    let mut b = b;
    let mut d = 1;

    while a&1==0 && b&1==0 {
        a >>= 1;
        b >>= 1;
        d <<= 1;
    }
    while a&1==0 {
        a>>=1;
    }
    while b&1==0 {
        b>>=1;
    }
    println!("{} {}",a,b);
    while a!=b {
        if a>b {
            a = a-b;
            while a&1==0 {
                a>>=1;
            }
        } else {
            b = b-a;
            while b&1==0 {
                b>>=1;
            }
        }
    }

    a*d
}
```

# 2024-08-22
[476. Number Complement](https://leetcode.com/problems/number-complement/)

```Rust
impl Solution {
    pub fn find_complement(num: i32) -> i32 {
        let mut a:i64 = 2;
        while a <= num as i64 {
            a *= 2;
        }
        a -= 1;

        (a-num as i64) as i32
    }
}
```

# 2024-08-21
[664. Strange Printer](https://leetcode.com/problems/strange-printer/)

This one is a bit difficult...


## Draft
```Rust
impl Solution {
    pub fn strange_printer(s: String) -> i32 {
        let mut s = s.as_bytes().to_vec();
        s.dedup();
        //println!("{:?}",std::str::from_utf8(&s).unwrap());
        s = s.into_iter().map(|x| x-b'a').collect();

        dfs(&s)
    }
}

fn dfs(s: &Vec<u8>) -> i32 {
    println!("{:?}",std::str::from_utf8(&s.iter().map(|x| (x+b'a')).collect::<Vec<u8>>()).unwrap());
    if s.len() <= 2 {
        return s.len() as i32;
    }
    let mut count = vec![0; 26];
    let mut t = s.clone();
    let mut rem = 0;
    loop {
        t.iter().for_each(|&x| count[x as usize]+=1);
        let c = count.iter().filter(|&&x| x==1).count();
        if c == 0 {
            break;
        }
        rem += c as i32;
        t = t.into_iter().filter(|&x| count[x as usize]!=1).collect::<Vec<u8>>();
        t.dedup();
        for i in 0..count.len() {
            count[i] = 0;
        }
    }
    println!("{} {:?}",rem, count);
    println!("{:?}",std::str::from_utf8(&t.iter().map(|x| (x+b'a')).collect::<Vec<u8>>()).unwrap());
    let s = t;
    if s.len() <= 2 {
        return s.len() as i32 + rem;
    }
    
    /*
    let mut a = Vec::new();
    a.push(0 as usize);
    for i in 1..count.len(){
        if count[*a.last().unwrap()] <= count[i]  {
            a.push(i);
        }
    }
    //println!("{:?}",a);
    let mut b = Vec::new();
    b.push(a.pop().unwrap());
    while !a.is_empty() && count[*b.last().unwrap()] == count[*a.last().unwrap()] {
        b.push(a.pop().unwrap());
    }
    //println!("{:?}",b);
    if count[*b.last().unwrap()] == 1 {
        return b.len() as i32;
    }

    let mut min = s.len() as i32;
    for i in b {
        let mut sum = 1;
        for e in s.split(|&x| x as usize==i) {
            sum += dfs(&e.to_vec());
        }
        if min > sum {
            min = sum;
        }
    }
    */

    let mut min = s.len() as i32;
    for i in 0..count.len() {
        if count[i] == 0 {
            continue;
        }
        let mut sum = 1;
        for e in s.split(|&x| x as usize==i) {
            sum += dfs(&e.to_vec());
        }
        if min > sum {
            min = sum;
        }
    }

    rem+min
}
```
# 2024-08-20
[1140. Stone Game II](https://leetcode.com/problems/stone-game-ii/)

A really novel problem... Had a good think. The problem is a bit mind boggling.
## Final
```Rust
impl Solution {
    pub fn stone_game_ii(piles: Vec<i32>) -> i32 {
        let mut scores = vec![vec![(0,0); piles.len()]; piles.len()];

        let mut cumsum = vec![0; piles.len()+1];
        for i in 0..piles.len() {
            cumsum[i+1] = cumsum[i] + piles[i];
        }

        dfs(&cumsum, &mut scores, 0, 1);
        
        /*
        for e in scores.iter() {
            println!("{:3?}",e);
        }
        */
        

        scores[0][0].0
    }
}

fn dfs(cumsum: &Vec<i32>, scores: &mut Vec<Vec<(i32,usize)>>, i: usize, m: usize) -> (i32,usize) {
    if i >= scores.len() {
        return (0,0);
    }
    if scores[i][m-1].0 != 0 {
        return scores[i][m-1];
    }
    let mut cans = Vec::new();

    let mm = if m*2 <= scores.len()-i {
        m*2
    } else {
        scores.len()-i
    };

    for j in 1..=mm {
        let ma = if j>m {
            j
        } else {
            m
        };
        let (val2, m2) = dfs(cumsum, scores, i+j, ma);


        let mb = if m2>ma {
            m2
        } else {
            ma
        };
        let (val3, m3) = dfs(cumsum, scores, i+j+m2, mb);

        let val = cumsum[i+j] - cumsum[i] + val3;

        cans.push((val,j));
    }
    scores[i][m-1] = *cans.iter().max().unwrap();

    scores[i][m-1]
}
```

## Draft 2
```Rust
impl Solution {
    pub fn stone_game_ii(piles: Vec<i32>) -> i32 {
        match piles.len() {
            1 => return piles[0],
            2 => return piles[0]+piles[1],
            _ => ()
        }

        let mut scores = vec![vec![(0,0); piles.len()]; piles.len()];
        
        let mut cumsum = vec![0; piles.len()+1];
        for i in 0..piles.len() {
            cumsum[i+1] = cumsum[i] + piles[i];
        }

        for i in (0..scores.len()).rev() {
            for j in 0..scores[i].len()-i {
                let mut v = cumsum[i+j+1]-cumsum[i];
                //print!("{} ",v);
                if i+j > scores.len()-3 {
                    scores[i][j] = (v,j+1);
                    if j>0 && scores[i][j].0 < scores[i][j-1].0 {
                        //scores[i][j] = scores[i][j-1];
                    }
                    continue;
                }

                /*
                let mut n = (j+1)*2;
                if n >= scores.len() {
                    n = scores.len();
                }
                let mut maxi = 0;
                let mut maxv = scores[i+j+1][0].0;
                for k in 1..n {
                    if scores[i+j+1][k].0 > maxv {
                        maxv = scores[i+j+1][k].0;
                        maxi = k;
                    }
                }
                */
                let mut m = (j+1)*2;
                if m >= scores.len() {
                    m = scores.len();
                }
                let mut max = scores[i+j+1].iter().take(m).max().unwrap().clone();

                if i+j+1+max.1 >= scores.len() {
                    scores[i][j] = (v,j+1);
                    if j>0 && scores[i][j].0 < scores[i][j-1].0 {
                        //scores[i][j] = scores[i][j-1];
                    }
                    continue;
                }

                if max.1*2 > m {
                    m = max.1*2;
                }
                if m >= scores.len() {
                    m = scores.len();
                }
                let mut max2 = scores[i+j+1+max.1].iter().take(m).max().unwrap();
                v += max2.0;

                /*
                if i+j+2+maxi >= scores.len() {
                    scores[i][j] = (v,j+1);
                    continue;
                }
                */
                
                /*
                if (maxi+1)*2 > n {
                    n = (maxi+1)*2;
                }
                */
                /*
                n = if j < maxi {
                    (j+1)*2
                } else {
                    (maxi+1)*2
                };
                if n >= scores.len() {
                    n = scores.len()-1;
                }
                let mut maxi2 = 0;
                let mut maxv2 = scores[i+j+2+maxi][0].0;
                for k in 1..n {
                    if scores[i+j+2+maxi][k].0 > maxv2 {
                        maxv2 = scores[i+j+2+maxi][k].0;
                        maxi2 = k;
                    }
                }
                v += maxv2;
                */

                scores[i][j] = (v,j+1);
                //scores[i][j] = *scores[i].iter().take(j+1).max().unwrap();
                if j>0 && scores[i][j].0 < scores[i][j-1].0 {
                    //println!("{:?} {:?}",scores[i][j],scores[i][j-1]);
                    //scores[i][j] = scores[i][j-1];
                }
            }
            println!("{:3?}",scores[i]);
        }

        scores[0].iter().take(2).max().unwrap().0
    }
}
```

## Draft
```Rust
impl Solution {
    pub fn stone_game_ii(piles: Vec<i32>) -> i32 {
        let mut score = vec![vec![0; piles.len()]; piles.len()];
        let mut cumsum = vec![0; piles.len()+1];

        for i in 0..piles.len() {
            cumsum[i+1] = cumsum[i] + piles[i];
        }

        dfs(&cumsum, &mut score, 1, 0);
        dfs(&cumsum, &mut score, 2, 0);
        println!("{:?}",score);
        
        *score[0].iter().max().unwrap()
    }
}

fn dfs(cumsum: &Vec<i32>, score: &mut Vec<Vec<i32>>, m: usize, i: usize) -> i32 {
    if i > cumsum.len()-1 {
        return 0;
    }
    if i+m > cumsum.len()-1 {
        return 0;
    }
    if score[i][m-1] != 0 {
        return score[i][m-1];
    }

    for j in 1..=m {
        let mut s = cumsum[i+j] - cumsum[i];
        println!("{}",s);
        let mut max = dfs(cumsum, score, 1, i+j);
        let mut maxk = 1;
        for k in 2..=j*2 {
            let t = dfs(cumsum, score, k, i+j);
            print!("{} ",t);
            if t >= max {
                max = t;
                maxk = k;
            }
        }
        println!();
        max = dfs(cumsum, score, maxk, i+j+maxk);
        let mut maxl = 1;
        for l in 2..=maxk*2 {
            let t = dfs(cumsum, score, l, i+j+maxk);
            if t >= max {
                max = t;
                maxl = l;
            }
        }
        s += dfs(cumsum, score, maxl, i+j+maxl);
        score[i][j-1] = s;
    }

    score[i][m-1]
}
```
# 2024-08-19
[650. 2 Keys Keyboard](https://leetcode.com/problems/2-keys-keyboard/)

Prime factorization problem.

```Rust
impl Solution {
    pub fn min_steps(n: i32) -> i32 {
        let p = vec![2,3,5,7,11,13,17,19,23,29,31];
        let mut n = n;
        let mut s = 0;
        let mut i = 0;

        while i<p.len() {
            while n%p[i] == 0 {
                n = n/p[i];
                s += p[i];
            }
            i += 1;
        }
        if n != 1 {
            s += n;
        }

        s
    }
}
```

# 2024-08-18
[264. Ugly Number II](https://leetcode.com/problems/ugly-number-ii/)

Works. Had to use 64-bit integers. Somehow the answer overflows turning the result negative. Surprisingly simple and manages avoid duplicate values.

Looking at it further, values that overflow get pushed onto the heap. Then popping the next value would give that overflow value??

The fastest solution is crazy... I suppose they keep track of the next possible value by using the current value multiplied with 2, 3, or 5 only if the next values match... They also keep an index of the current value to index into the calculated values...

## Solution
```Rust
use std::collections::BinaryHeap;

impl Solution {
    pub fn nth_ugly_number(n: i32) -> i32 {
        if n < 7 {
            return n;
        }
        
        let mut heap = BinaryHeap::new();
        heap.push((-2 as i64,2));
        heap.push((-3,3));
        heap.push((-5,5));
        let mut i = 2;
        while i < n {
            let (val,p) = heap.pop().unwrap();
            match p {
                2 => {
                    heap.push((val*2,2));
                    heap.push((val*3,3));
                    heap.push((val*5,5));
                }
                3 => {
                    heap.push((val*3,3));
                    heap.push((val*5,5));
                }
                5 => {
                    heap.push((val*5,5));
                }
                _ => ()
            }
            i += 1;
        }
        
        -heap.pop().unwrap().0 as i32
    }
}
```

## Draft Snippet
```Rust
let mut i = 7;
let mut heap = BinaryHeap::new();
//heap.push((-1,0,0,0));
heap.push((-8,3,0,0));
heap.push((-9,0,2,0));
heap.push((-10,1,0,1));
heap.push((-12,2,1,0));
heap.push((-15,0,1,1));
heap.push((-18,1,2,0));
heap.push((-20,2,0,1));
heap.push((-25,0,0,2));
heap.push((-30,1,1,1));
while i < n {
    let (val,ii,iii,v) = heap.pop().unwrap();
    print!("{} ",-val);
    if ii>iii && iii>v {
        heap.push((val*2,ii+1,iii,v));
        heap.push((val*3,ii,iii+1,v));
        heap.push((val*5,ii,iii,v+1));
    } else if iii>v {
        heap.push((val*3,ii,iii+1,v));
        heap.push((val*5,ii,iii,v+1));
    } else {
        heap.push((val*5,ii,iii,v+1));
    }
    i += 1;
}
```

# 2024-08-17
[1937. Maximum Number of Points with Cost](https://leetcode.com/problems/maximum-number-of-points-with-cost/)

## Slow but Passes
```Rust
impl Solution {
    pub fn max_points(points: Vec<Vec<i32>>) -> i64 {
        let e = &points[0];

        let mut a = Vec::new();
        let mut v = e[0];
        a.push((e[0] as i64, 0));

        for i in 1..e.len() {
            v -= 1;
            if e[i] >= v {
                a.push((e[i] as i64, i));
                v = e[i];
            }
        }

        let mut b = Vec::new();
        b.push(a.pop().unwrap());

        while !a.is_empty() {
            let c = a.pop().unwrap();
            let d = b.last().unwrap();

            if c.0 >= d.0 - d.1 as i64 + c.1 as i64 {
                b.push(c);
            }
        }
        let mut curr = b;

        //println!("{:?}",curr);

        for i in 1..points.len() {
            let mut a = Vec::new();
            for j in 0..points[i].len() {
                let mut max = 0;
                for e in curr.iter() {
                    let mut d = j as i64 - e.1 as i64;
                    if d<0 {
                        d = -d;
                    }
                    let t = points[i][j] as i64 + e.0 - d;
                    if t > max {
                        max = t;
                    }
                }
                a.push((max,j));
            }
            //println!("**{:?}",a);
            let mut b = Vec::new();
            let mut m = (0,0);
            for e in a {
                if b.is_empty() {
                    b.push(e);
                    m = e;
                } else {
                    if m.0 + (m.1 - e.1) as i64 <= e.0 {
                        b.push(e);
                        m = e;
                    }
                }
            }
            //println!("*{:?}",b);
            let mut c = Vec::new();
            let mut m = (0,0);
            for e in b.into_iter().rev() {
                if c.is_empty() {
                    c.push(e);
                    m = e;
                } else {
                    if m.0 + (e.1 - m.1) as i64 <= e.0 {
                        c.push(e);
                        m = e;
                    }
                }
            }
            //println!("*{:?}",c);
            curr = c;
        }

        curr.iter().map(|x| x.0).max().unwrap()
    }
}

```

## Second Try
```Rust
impl Solution {
    pub fn max_points(points: Vec<Vec<i32>>) -> i64 {
        let mut best = Vec::new();

        for e in points.iter() {
            let mut a = Vec::new();
            let mut v = e[0];
            a.push((e[0],0,0));

            for i in 1..e.len() {
                v -= 1;
                if e[i] >= v {
                    a.push((e[i],i as i32,0 as i64));
                    v = e[i];
                }
            }
            //println!("{:?}",a);

            let mut b = Vec::new();
            b.push(a.pop().unwrap());

            while !a.is_empty() {
                let c = a.pop().unwrap();
                let d = b.last().unwrap();

                if c.0 >= d.0-d.1+c.1 {
                    b.push(c);
                }
            }
            //println!("{:?}",b);
            best.push(b);
        }

        //println!("{:?}",best);

        for i in 0..best[0].len() {
            best[0][i].2 = best[0][i].0 as i64;
        }

        for i in 1..best.len() {
            for j in 0..best[i].len() {
                for k in 0..best[i-1].len() {
                    let mut a = (best[i][j].1 - best[i-1][k].1) as i64;
                    if a < 0 {
                        a = -a;
                    }
                    if best[i][j].0 as i64 + best[i-1][k].2 - a > best[i][j].2 {
                        best[i][j].2 = best[i][j].0 as i64 + best[i-1][k].2 - a;
                    }
                }
            }
        }

        println!("{:?}",best);
        
        best[best.len()-1].iter().map(|x| x.2).max().unwrap()
    }
}
```
## Brute Force (TLE)
```Rust
impl Solution {
    pub fn max_points(points: Vec<Vec<i32>>) -> i64 {
        let mut sum = points[0].iter().map(|x| *x as i64).collect::<Vec<i64>>();

        for i in 1..points.len() {
            let mut num = Vec::new();
            for j in 0..points[i].len() {
                let mut max = 0;
                for k in 0..j {
                    let t = sum[k] + points[i][j] as i64 + (k-j) as i64;
                    if t>max{
                        max = t;
                    }
                }
                for k in j..points[i].len() {
                    let t = sum[k] + points[i][j] as i64 - (k-j) as i64;
                    if t>max{
                        max = t;
                    }
                }
                num.push(max);
            }
            sum = num;
            //println!("{:?}",sum);
        }
        println!("{:?}",sum);
        sum.into_iter().max().unwrap()
    }
}
```
# 2024-08-16
[624. Maximum Distance in Arrays](https://leetcode.com/problems/maximum-distance-in-arrays/)

Basically keeping track of two minimums and two maximums.

## Faster and Less Memory
```Rust
use std::collections::BinaryHeap;

impl Solution {
    pub fn max_distance(arrays: Vec<Vec<i32>>) -> i32 {
        let mut min = BinaryHeap::new();
        let mut max = BinaryHeap::new();

        for i in 0..arrays.len() {
            min.push((arrays[i][0],i));
            max.push((-arrays[i][arrays[i].len()-1],i));
            if min.len() > 2 {
                min.pop();
            }
            if max.len() > 2 {
                max.pop();
            }
        }

        let min1 = min.pop().unwrap();
        let max1 = max.pop().unwrap();
        let min2 = min.pop().unwrap();
        let max2 = max.pop().unwrap();
        if max2.1 == min2.1 {
            if min1.0+max2.0 < min2.0+max1.0 {
                -(min1.0+max2.0)
            } else {
                -(min2.0+max1.0)
            }
        } else {
            -(max2.0 + min2.0)
        }
    }
}
```
## Works
```Rust
use std::collections::BinaryHeap;

impl Solution {
    pub fn max_distance(arrays: Vec<Vec<i32>>) -> i32 {
        let mut min = BinaryHeap::new();
        let mut max = BinaryHeap::new();

        for i in 0..arrays.len() {
            min.push((-arrays[i][0],i));
            max.push((arrays[i][arrays[i].len()-1],i));
        }

        let min1 = min.pop().unwrap();
        let max1 = max.pop().unwrap();
        if max1.1 == min1.1 {
            let min2 = min.pop().unwrap();
            let max2 = max.pop().unwrap();
            if min1.0+max2.0 > min2.0+max1.0 {
                min1.0+max2.0
            } else {
                min2.0+max1.0
            }
        } else {
            max1.0 + min1.0
        }
    }
}
```
# 2024-08-15
[860. Lemonade Change](https://leetcode.com/problems/lemonade-change/)

Easy peasy.

```Rust
impl Solution {
    pub fn lemonade_change(bills: Vec<i32>) -> bool {
        let mut bv = 0;
        let mut bx = 0;
        //let mut bxx = 0;
        for b in bills.iter() {
            match b {
                5 => bv += 1,
                10 => {
                    if bv > 0 {
                        bv -= 1;
                        bx += 1;
                    } else {
                        return false;
                    }
                }
                20 => {
                    if bx>0 && bv>0 {
                        bv -= 1;
                        bx -= 1;
                        //bxx += 1;
                    } else if bv > 2 {
                        bv -= 3;
                        //bxx += 1;
                    } else {
                        return false;
                    }
                }
                _ => ()
            }
        }
        true
    }
}
```
# 2024-08-14
[719. Find K-th Smallest Pair Distance](https://leetcode.com/problems/find-k-th-smallest-pair-distance/)

The problem suggests using Binary Search. I'm really skeptical if it would be faster.

## Brute Force
```Rust
impl Solution {
    pub fn smallest_distance_pair(nums: Vec<i32>, k: i32) -> i32 {
        let mut nums = nums;
        nums.sort_unstable();
        let mut dist = Vec::new();
        for i in 0..nums.len() {
            for j in i+1..nums.len() {
                dist.push(nums[j]-nums[i]);
            }
        }
        dist.sort_unstable();
        dist[k as usize-1]
    }
}
```
# 2024-08-13
[40. Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)

There should be a faster way. DFS does have redundancies...
## Naive Approach
```Rust
impl Solution {
    pub fn combination_sum2(candidates: Vec<i32>, target: i32) -> Vec<Vec<i32>> {
        let mut a: Vec<Vec<i32>> = Vec::new();
        let mut b = Vec::new();
        let mut candidates = candidates;
        
        candidates.sort_unstable();
        //println!("{:?}",candidates);
        let mut last = 0;

        for i in 0..candidates.len() {
            if last == candidates[i] {
                continue;
            }
            last = candidates[i];
            dfs(&candidates, &mut a, &mut b, target, i);
        }
        a
    }
}

fn dfs(cans: &Vec<i32>, combos: &mut Vec<Vec<i32>>, curr: &mut Vec<i32>, target: i32, index: usize) {
    if index >= cans.len() {
        return;
    }
    if target < cans[index] {
        dfs(cans,combos,curr,target,index+1);
        return;
    }
    if target == cans[index] {
        curr.push(cans[index]);
        combos.push(curr.clone());
        curr.pop();
        return;
    }
    
    curr.push(cans[index]);
    let target = target-cans[index];
    let mut last = 0;
    for i in index+1..cans.len() {
        if last == cans[i] {
            continue;
        }
        last = cans[i];
        dfs(cans,combos,curr,target,i);
    }
    curr.pop();
}
```
# 2024-08-12
[703. Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/)

I was trying to optimize it... But then I read the top solution and had a "got it" moment. Dang, I should have figured it out...

Essentially, you only need to keep track of k largest elements.

## Optimized
```Rust
use std::collections::BinaryHeap;

struct KthLargest {
    k: i32,
    heap: BinaryHeap<i32>,
}


/** 
 * `&self` means the method takes an immutable reference.
 * If you need a mutable reference, change it to `&mut self` instead.
 */
impl KthLargest {
    fn new(k: i32, nums: Vec<i32>) -> Self {
        let mut heap = BinaryHeap::new();
        for e in nums.iter() {
            heap.push(-e);
        }
        while heap.len() > (k as usize) {
            heap.pop();
        }
        KthLargest {k,heap}
    }
    
    fn add(&mut self, val: i32) -> i32 {
        self.heap.push(-val);
        while self.heap.len() > (self.k as usize) {
            self.heap.pop();
        }
        0-self.heap.peek().unwrap()
    }
}

/**
 * Your KthLargest object will be instantiated and called as such:
 * let obj = KthLargest::new(k, nums);
 * let ret_1: i32 = obj.add(val);
 */
```

## Unoptimized
```Rust
struct KthLargest {
    k: i32,
    nums: Vec<i32>,
}


/** 
 * `&self` means the method takes an immutable reference.
 * If you need a mutable reference, change it to `&mut self` instead.
 */
impl KthLargest {
    fn new(k: i32, nums: Vec<i32>) -> Self {
        let mut nums = nums;
        //nums = nums.iter().map(|x| 0-x).collect::<Vec<i32>>();
        nums.sort_unstable();
        KthLargest {
            k,
            nums: nums,
        }
    }
    
    fn add(&mut self, val: i32) -> i32 {
        //self.nums.push(val);
        //self.nums.sort();
        //let val = 0-val;
        match self.nums.binary_search(&val) {
            Ok(pos) => self.nums.insert(pos,val),
            Err(pos) => self.nums.insert(pos,val),
        }
        self.nums[self.nums.len()-self.k as usize]
        //0-self.nums[(self.k-1) as usize]
    }
}

/**
 * Your KthLargest object will be instantiated and called as such:
 * let obj = KthLargest::new(k, nums);
 * let ret_1: i32 = obj.add(val);
 */
```
# 2024-08-11
[1568. Minimum Number of Days to Disconnect Island](https://leetcode.com/problems/minimum-number-of-days-to-disconnect-island/)

The minimum is at most 4 moves. We can take any arbitrary land cell and make its neighbors into water cells.

Took a lot of thinking but at least solved it. Various edge cases need to be dealt with.

First test is to check connectivity to make sure there is only a single group of cells.

The second test looks at the number of cells. 0 cells = 0, 1 cell = 1, 2 cells = 2, 3 cells = 1. This will always be true given the cells are connected.

The third test forms the core of this algorithm. The idea is to find the shortest path between two cells. This can be done by calculating the distance from each of the two cells then summing up both distances together. The distance between the two cells represents the shortest possible distance (BFS). Thus if the sum of two distances is equal to the shortest distance, the cell is on the shortest path.

From there we take a histogram of the (unsummed) distances of the cells on the shortest path. If we get at least three 1's, then we only need to remove one cell.

Otherwise we have to remove two cells. (At max we only need to remove 2 cells to create two (or zero) sets of cells.)

The summing technique is reminiscent of [2045. Second Minimum Time to Reach Destination](https://leetcode.com/problems/second-minimum-time-to-reach-destination/). The summing technique essentially finds all possible shortest paths from any two nodes.


```Rust
impl Solution {
    pub fn min_days(grid: Vec<Vec<i32>>) -> i32 {
        /*
        if grid.len()>1 && grid[0].len()>1
            && grid[0]==[1,1] && grid[1]==[1,0] {
            return 1;
        }
        */

        let mut first = true;
        let mut first_cell = (0,0);
        let mut last_cell = (0,0);
        let mut num_cells = 0;

        let mut costa = vec![vec![-1; grid[0].len()]; grid.len()];
        for i in 0..grid.len() {
            for j in 0..grid[0].len() {
                let c = bfs(&grid,&mut costa,(i,j));
                if c>0 {
                    if first {
                        first = false;
                        first_cell = (i,j);
                        num_cells = c;
                    } else {
                        return 0;
                    }
                    //last_cell = (i,j);
                }
            }
        }

        if num_cells == 0 {
            return 0;
        }
        if num_cells == 2 {
            return 2;
        }
        if num_cells < 4 {
            return 1;
        }

        for i in 0..grid.len() {
            for j in 0..grid[0].len() {
                if grid[i][j]==1 && num_neighbors(&grid,(i,j))==1 {
                    return 1;
                }
            }
        }

        let mut max = 0;
        for i in 0..grid.len() {
            for j in 0..grid[0].len() {
                if costa[i][j] > max {
                    max = costa[i][j];
                    last_cell = (i,j);
                }
            }
        }

        let mut costb = vec![vec![-1; grid[0].len()]; grid.len()];
        bfs(&grid,&mut costb,last_cell);

        let mut costc =
            costa.iter().zip(costb.iter())
            .map(|(a,b)| a.iter().zip(b.iter())
                .map(|(c,d)| c+d)
                .collect::<Vec<i32>>())
            .collect::<Vec<_>>();

        let critval = costc[first_cell.0][first_cell.1];
        let mut count = vec![0; (critval+1) as usize];
        for i in 0..costc.len() {
            for j in 0..costc[0].len() {
                if costc[i][j] == critval {
                    count[costa[i][j] as usize] += 1;
                }
            }
        }

        /*
        for e in costa.iter() {
            println!("{:2?}",e);
        }
        println!();
        for e in costb.iter() {
            println!("{:2?}",e);
        }
        println!();
        for e in costc.iter() {
            println!("{:2?}",e);
        }
        println!("first: {:?}; last: {:?}",first_cell,last_cell);
        println!("count: {:?}", count);
        */

        if count.iter().filter(|&&e| e==1).count() > 2 {
            return 1;
        }

        return 2;
    }
}

/*
fn dfs(grid: & Vec<Vec<i32>>, cost: &mut Vec<Vec<i32>>, start: (usize,usize), val: i32) -> i32 {
    if start.0 >= grid.len() {
        return 0;
    }
    if start.1 >= grid[0].len() {
        return 0;
    }
    if grid[start.0][start.1] == 0 {
        return 0;
    }
    if cost[start.0][start.1] >= 0 {
        return 0;
    }

    let mut c = 1;

    cost[start.0][start.1] = val;

    c += dfs(grid, cost, (start.0-1, start.1), val+1); // up
    c += dfs(grid, cost, (start.0, start.1-1), val+1); // left
    c += dfs(grid, cost, (start.0, start.1+1), val+1); // right
    c += dfs(grid, cost, (start.0+1, start.1), val+1); // down

    return c;
}
*/

fn bfs(grid: & Vec<Vec<i32>>, cost: &mut Vec<Vec<i32>>, cell: (usize,usize)) -> i32 {
    let mut val = 0;
    
    let mut a = Vec::new();
    a.push(cell);

    let mut c = 0;

    while !a.is_empty() {
        let mut b = Vec::new();
        while !a.is_empty() {
            match a.pop() {
                None => (),
                Some(e) => {
                    if set_val(grid,cost,e,val) {
                        c += 1;
                        b.push((e.0-1,e.1));
                        b.push((e.0+1,e.1));
                        b.push((e.0,e.1+1));
                        b.push((e.0,e.1-1));
                    }
                }
            }
        }
        val += 1;
        a = b;
    }

    c
}

fn set_val(grid: & Vec<Vec<i32>>, cost: &mut Vec<Vec<i32>>, cell: (usize,usize), val: i32) -> bool {
    if cell.0 >= grid.len() {
        return false;
    }
    if cell.1 >= grid[0].len() {
        return false;
    }
    if grid[cell.0][cell.1] == 0 {
        return false;
    }
    if cost[cell.0][cell.1] >= 0 {
        return false;
    }

    cost[cell.0][cell.1] = val;

    return true;
}

fn num_neighbors(grid: & Vec<Vec<i32>>, cell: (usize,usize)) -> i32 {
    let mut a = Vec::new();
    if cell.0 > 0 {
        a.push((cell.0-1,cell.1));
    }
    if cell.1 > 0 {
        a.push((cell.0,cell.1-1));
    }
    if cell.0 < grid.len()-1 {
        a.push((cell.0+1,cell.1));
    }
    if cell.1 < grid[0].len()-1 {
        a.push((cell.0,cell.1+1));
    }

    let mut c = 0;
    for e in a {
        c += grid[e.0][e.1];
    }
    c
}
```

# 2024-08-10
[959. Regions Cut By Slashes](https://leetcode.com/problems/regions-cut-by-slashes/)

Nothing too crazy here. A very basic implementation of union-find.

Tried implementing "path compression" but it doesn't seem to be any faster... At least not for "small" sizes... (n~900).
## Find Compression
```Rust
fn union(parent: &mut Vec<usize>, a: usize, b: usize) -> usize {
    let a = find(parent, a);
    let b = find(parent, b);
    if a < b {
        parent[b] = a;
        a
    } else {
        parent[a] = b;
        b
    }
}

fn find(parent: &mut Vec<usize>, a: usize) -> usize {
    let mut r = a;
    while r != parent[r] {
        r = parent[r];
    }
    let mut a = a;
    while a != parent[a] {
        let t = parent[a];
        parent[a] = r;
        a = t;
    }
    r
}
```

## Clean

```Rust
impl Solution {
    pub fn regions_by_slashes(grid: Vec<String>) -> i32 {
        let mut down  = vec![vec![0; grid[0].len()]; grid.len()];
        let mut right = vec![vec![0; grid[0].len()]; grid.len()];

        let mut parent = Vec::new();
        parent.push(0);
        let mut c = 1;
        
        for i in 0..grid.len() {
            for j in 0..grid[i].len() {
                match grid[i].as_bytes()[j] {
                    b'/' => {
                        if i==0 && j==0 {
                            parent.push(c);
                            c += 1;
                        } else if i!=0 && j!=0 {
                            let u = down[i-1][j];
                            let l = right[i][j-1];
                            union(&mut parent, u, l);
                        }
                        down[i][j] = c;
                        right[i][j] = c;
                        parent.push(c);
                        c += 1;
                    },
                    b'\\' => {
                        if i==0 {
                            right[i][j] = c;
                            parent.push(c);
                            c += 1;
                        } else {
                            right[i][j] = down[i-1][j];
                        }
                        if j==0 {
                            down[i][j] = c;
                            parent.push(c);
                            c += 1;
                        } else {
                            down[i][j] = right[i][j-1];
                        }
                    },
                    b' ' => {
                        if i==0 && j==0 {
                            right[i][j] = c;
                            down[i][j] = c;
                            parent.push(c);
                            c += 1;
                        } else if i==0 {
                            right[i][j] = right[i][j-1];
                            down[i][j] = right[i][j-1];
                        } else if j==0 {
                            right[i][j] = down[i-1][j];
                            down[i][j] = down[i-1][j];
                        } else {
                            let u = down[i-1][j];
                            let l = right[i][j-1];
                            let a = union(&mut parent, u, l);
                            right[i][j] = a;
                            down[i][j] = a;
                        }
                    },
                    _ => ()
                }
            }
        }

        let mut out = 0;
        for i in 1..parent.len() {
            if i == parent[i]  {
                out += 1;
            }
        }
        out
    }
}

fn union(parent: &mut Vec<usize>, a: usize, b: usize) -> usize {
    let mut a = a;
    let mut b = b;
    while a != parent[a] {
        a = parent[a];
    }
    while b != parent[b] {
        b = parent[b];
    }
    if a < b {
        parent[b] = a;
        a
    } else {
        parent[a] = b;
        b
    }
}
```
## Draft

```Rust
impl Solution {
    pub fn regions_by_slashes(grid: Vec<String>) -> i32 {
        //let mut up    = vec![vec![0; grid[0].len()]; grid.len()];
        let mut down  = vec![vec![0; grid[0].len()]; grid.len()];
        //let mut left  = vec![vec![0; grid[0].len()]; grid.len()];
        let mut right = vec![vec![0; grid[0].len()]; grid.len()];

        let mut parent = Vec::new();
        parent.push(0);
        let mut c = 1;
        
        for i in 0..grid.len() {
            for j in 0..grid[i].len() {
                match grid[i].as_bytes()[j] {
                    b'/' => {
                        if i==0 && j==0 {
                            //up[i][j] = c;
                            //left[i][j] = c;
                            parent.push(c);
                            c += 1;
                        } else if i==0 {
                            //left[i][j] = right[i][j-1];
                            //up[i][j] = right[i][j-1];
                        } else if j==0 {
                            //up[i][j] = down[i-1][j];
                            //left[i][j] = down[i-1][j];
                        } else {
                            let u = down[i-1][j];
                            let l = right[i][j-1];
                            /*
                            if u<l {
                                parent[l] = u;
                            } else {
                                parent[u] = l;
                            }
                            */
                            union(&mut parent, u, l);
                        }
                        down[i][j] = c;
                        right[i][j] = c;
                        parent.push(c);
                        c += 1;
                    },
                    b'\\' => {
                        if i==0 {
                            right[i][j] = c;
                            parent.push(c);
                            c += 1;
                        } else {
                            right[i][j] = down[i-1][j];
                        }
                        if j==0 {
                            down[i][j] = c;
                            parent.push(c);
                            c += 1;
                        } else {
                            down[i][j] = right[i][j-1];
                        }
                    },
                    b' ' => {
                        if i==0 && j==0 {
                            right[i][j] = c;
                            down[i][j] = c;
                            parent.push(c);
                            c += 1;
                        } else if i==0 {
                            right[i][j] = right[i][j-1];
                            down[i][j] = right[i][j-1];
                        } else if j==0 {
                            right[i][j] = down[i-1][j];
                            down[i][j] = down[i-1][j];
                        } else {
                            let u = down[i-1][j];
                            let l = right[i][j-1];
                            let a = union(&mut parent, u, l);
                            right[i][j] = a;
                            down[i][j] = a;
                        }
                    },
                    _ => ()
                }
            }
        }

        /*
        println!("{:?}",parent);
        println!("{:?}",right);
        println!("{:?}",down);
        */

        let mut out = 0;

        for i in 1..parent.len() {
            /*
            let mut j = i;
            while j != parent[j] {
                j = parent[j];
            }
            */
            if i == parent[i]  {
                out += 1;
            }
        }

        out
    }
}

fn union(parent: &mut Vec<usize>, a: usize, b: usize) -> usize {
    let mut a = a;
    let mut b = b;
    while a != parent[a] {
        a = parent[a];
    }
    while b != parent[b] {
        b = parent[b];
    }
    if a < b {
        parent[b] = a;
        a
    } else {
        parent[a] = b;
        b
    }
}
```
# 2024-08-09
[840. Magic Squares In Grid](https://leetcode.com/problems/magic-squares-in-grid/)

No weird cumulative sum stuff. Just keeping it simple. In particular with sums of 3 numbers, I don't think there's any benefits. (It's still a sum/difference of 3 numbers.)

Likely only one diagonal sum is needed and not both. One diagonal sum should be sufficient to "lock" the position of digits. Putting it another way, a diagonal sum (as well as all row sums and col sums) imply the other diagonal sum should be equal as well.

Also every sum should equal to 15. An intuitive explanation is that the total sum of 1 to 9 is 45... This gets split into 3 sums so each should equal 15.

Another property of magic squares could be that 5 must be at the center...

The other part is to check it's actually 1 to 9... Otherwise all 5s could be considered as a magic square...

```Rust
impl Solution {
    pub fn num_magic_squares_inside(grid: Vec<Vec<i32>>) -> i32 {
        if grid.len() < 3 {
            return 0;
        }
        if grid[0].len() < 3 {
            return 0;
        }
        let mut bitmap = vec![vec![0; grid[0].len()-2]; grid.len()-2];

        for i in 0..grid.len()-2 {
            for j in 0..grid[0].len()-2 {
                /*
                if i==0 {
                    if j==0 {
                        bitmap[i][j] =
                            (1<<grid[i  ][j]) | (1<<grid[i  ][j+1]) | (1<<grid[i  ][j+2]) |
                            (1<<grid[i+1][j]) | (1<<grid[i+1][j+1]) | (1<<grid[i+1][j+2]) |
                            (1<<grid[i+2][j]) | (1<<grid[i+2][j+1]) | (1<<grid[i+2][j+2]);
                    } else {
                        bitmap[i][j] = bitmap[i][j-1];
                        bitmap[i][j] &= !((1<<grid[i][j-1]) | (1<<grid[i+1][j-1]) | (1<<grid[i+2][j-1]));
                        bitmap[i][j] |=   (1<<grid[i][j+2]) | (1<<grid[i+1][j+2]) | (1<<grid[i+2][j+2]);
                    }
                } else {
                    bitmap[i][j] = bitmap[i-1][j];
                    bitmap[i][j] &= !((1<<grid[i-1][j]) | (1<<grid[i-1][j+1]) | (1<<grid[i-1][j+2]));
                    bitmap[i][j] |=   (1<<grid[i+2][j]) | (1<<grid[i+2][j+1]) | (1<<grid[i+2][j+2]);
                }
                */
                bitmap[i][j] =
                    (1<<grid[i  ][j]) | (1<<grid[i  ][j+1]) | (1<<grid[i  ][j+2]) |
                    (1<<grid[i+1][j]) | (1<<grid[i+1][j+1]) | (1<<grid[i+1][j+2]) |
                    (1<<grid[i+2][j]) | (1<<grid[i+2][j+1]) | (1<<grid[i+2][j+2]);
                //print!("{:b} ",bitmap[i][j]);
            }
            //println!();
        }

        let mut sumrow = vec![vec![0; grid[0].len()-2]; grid.len()];
        let mut sumcol = vec![vec![0; grid[0].len()]; grid.len()-2];
        let mut sumdia = vec![vec![0; grid[0].len()-2]; grid.len()-2];
        //let mut sumdia2 = vec![vec![0; grid[0].len()-2]; grid.len()-2];

        for i in 0..grid.len() {
            for j in 0..grid[0].len()-2 {
                sumrow[i][j] = grid[i][j] + grid[i][j+1] + grid[i][j+2];
            }
        }
        for i in 0..grid.len()-2 {
            for j in 0..grid[0].len() {
                sumcol[i][j] = grid[i][j] + grid[i+1][j] + grid[i+2][j];
            }
        }
        for i in 0..grid.len()-2 {
            for j in 0..grid[0].len()-2 {
                sumdia[i][j] = grid[i][j] + grid[i+1][j+1] + grid[i+2][j+2];
            }
        }
        /*
        for i in 0..grid.len()-2 {
            for j in 0..grid[0].len()-2 {
                sumdia2[i][j] = grid[i][j+2] + grid[i+1][j+1] + grid[i+2][j];
            }
        }
        */

        let mut count = 0;

        for i in 0..grid.len()-2 {
            for j in 0..grid[0].len()-2 {
                if bitmap[i][j] != 0b1111111110 {continue;}
                if sumrow[i][j] != 15 {continue;}
                if sumrow[i+1][j] != 15 {continue;}
                if sumrow[i+2][j] != 15 {continue;}
                if sumcol[i][j] != 15 {continue;}
                if sumcol[i][j+1] != 15 {continue;}
                if sumcol[i][j+2] != 15 {continue;}
                if sumdia[i][j] != 15 {continue;}
                //if sumdia2[i][j] != 15 {continue;}

                count += 1;
            }
        }

        count
    }
}
```
# 2024-08-08
[885. Spiral Matrix III](https://leetcode.com/problems/spiral-matrix-iii/)

Just simulating the pattern.

There are two patterns to encode:
- Right, Down, Left, Up.
- 1, 1, 2, 2, 3, 3, 4, 4, 5, 5, and so on...

From there it's keeping the positions that are within bounds and having a finish condition for the while loop.

```Rust
impl Solution {
    pub fn spiral_matrix_iii(rows: i32, cols: i32, r_start: i32, c_start: i32) -> Vec<Vec<i32>> {
        let mut pos = Vec::new();
        let mut curr = (r_start, c_start);
        let dirs = [(0,1),(1,0),(0,-1),(-1,0)];
        let mut dir = 0;
        let mut count = 1;
        let size = (rows*cols) as usize;

        pos.push([curr.0,curr.1].to_vec());
        while pos.len() < size {
            for i in 0..count {
                curr.0 += dirs[dir].0;
                curr.1 += dirs[dir].1;
                if curr.0<rows && curr.0>=0 && curr.1<cols && curr.1>=0 {
                    pos.push([curr.0,curr.1].to_vec());
                }
            }
            dir = (dir+1)%4;
            for i in 0..count {
                curr.0 += dirs[dir].0;
                curr.1 += dirs[dir].1;
                if curr.0<rows && curr.0>=0 && curr.1<cols && curr.1>=0 {
                    pos.push([curr.0,curr.1].to_vec());
                }
            }
            dir = (dir+1)%4;
            count += 1;
        }

        pos
    }
}
```
# 2024-08-07
[273. Integer to English Words](https://leetcode.com/problems/integer-to-english-words/)

Simple.

## Clean
```Rust
impl Solution {
    pub fn number_to_words(num: i32) -> String {
        if num==0 {
            return "Zero".to_string();
        }

        let mut s = "".to_string();

        let n = num/1_000_000_000;
        num_to_string(n, &mut s);
        if n>0 {
            s.push_str(" Billion");
            if num%1_000_000_000 > 0 {
                s.push_str(" ");
            }
        }

        let n = num/1_000_000%1000;
        num_to_string(n, &mut s);
        if n>0 {
            s.push_str(" Million");
            if num%1_000_000 > 0 {
                s.push_str(" ");
            }
        }

        let n = num/1_000%1000;
        num_to_string(n, &mut s);
        if n>0 {
            s.push_str(" Thousand");
            if num%1000 > 0 {
                s.push_str(" ");
            }
        }

        let n = num%1000;
        num_to_string(n, &mut s);

        s
    }
}

fn num_to_string(n: i32, s: &mut String) {
    let ones = ["", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine"];
    let teens = [
        "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen",
        "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"
    ];
    let tens = ["", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"];

    let h = n/100%10;
    let t = n/10%10;
    let o = n%10;
    if h > 0 {
        s.push_str(ones[h as usize]);
        s.push_str(" Hundred");
        if n%100 > 0 {
            s.push_str(" ");
        }
    }
    if t > 1 {
        s.push_str(tens[t as usize]);
        if o > 0 {
            s.push_str(" ");
            s.push_str(ones[o as usize]);
        }
    } else if t == 1 {
        s.push_str(teens[o as usize]);
    } else {
        s.push_str(ones[o as usize]);
    }
}
```
## Unfactored
```Rust
impl Solution {
    pub fn number_to_words(num: i32) -> String {
        if num==0 {
            return "Zero".to_string();
        }
        let ones = vec!["", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine"];
        let teens = vec![
            "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen",
            "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"
        ];
        let tens = vec!["", "Ten", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"];

        
        let mut s = "".to_string();

        let n = num/1_000_000_000;
        let h = n/100%10;
        let t = n/10%10;
        let o = n%10;
        if h > 0 {
            s.push_str(ones[h as usize]);
            s.push_str(" Hundred");
            if n%100 > 0 {
                s.push_str(" ");
            }
        }
        if t > 1 {
            s.push_str(tens[t as usize]);
            if o > 0 {
                s.push_str(" ");
                s.push_str(ones[o as usize]);
            }
        } else if t == 1 {
            s.push_str(teens[o as usize]);
        } else {
            s.push_str(ones[o as usize]);
        }
        if n>0 {
            s.push_str(" Billion");
            if num%1_000_000_000 > 0 {
                s.push_str(" ");
            }
        }

        let n = num/1_000_000%1000;
        let h = n/100%10;
        let t = n/10%10;
        let o = n%10;
        if h > 0 {
            s.push_str(ones[h as usize]);
            s.push_str(" Hundred");
            if n%100 > 0 {
                s.push_str(" ");
            }
        }
        if t > 1 {
            s.push_str(tens[t as usize]);
            if o > 0 {
                s.push_str(" ");
                s.push_str(ones[o as usize]);
            }
        } else if t == 1 {
            s.push_str(teens[o as usize]);
        } else {
            s.push_str(ones[o as usize]);
        }
        if n>0 {
            s.push_str(" Million");
            if num%1_000_000 > 0 {
                s.push_str(" ");
            }
        }

        let n = num/1_000%1000;
        let h = n/100%10;
        let t = n/10%10;
        let o = n%10;
        if h > 0 {
            s.push_str(ones[h as usize]);
            s.push_str(" Hundred");
            if n%100 > 0 {
                s.push_str(" ");
            }
        }
        if t > 1 {
            s.push_str(tens[t as usize]);
            if o > 0 {
                s.push_str(" ");
                s.push_str(ones[o as usize]);
            }
        } else if t == 1 {
            s.push_str(teens[o as usize]);
        } else {
            s.push_str(ones[o as usize]);
        }
        if n>0 {
            s.push_str(" Thousand");
            if num%1000 > 0 {
                s.push_str(" ");
            }
        }

        let n = num%1000;
        let h = n/100%10;
        let t = n/10%10;
        let o = n%10;
        if h > 0 {
            s.push_str(ones[h as usize]);
            s.push_str(" Hundred");
            if n%100 > 0 {
                s.push_str(" ");
            }
        }
        if t > 1 {
            s.push_str(tens[t as usize]);
            if o > 0 {
                s.push_str(" ");
                s.push_str(ones[o as usize]);
            }
        } else if t == 1 {
            s.push_str(teens[o as usize]);
        } else {
            s.push_str(ones[o as usize]);
        }

        s
    }
}

```
# 2024-08-06
[3016. Minimum Number of Pushes to Type Word II](https://leetcode.com/problems/minimum-number-of-pushes-to-type-word-ii/)

```Rust
impl Solution {
    pub fn minimum_pushes(word: String) -> i32 {
        let mut count = vec![0; 26];
        for c in word.bytes() {
            count[(c-b'a') as usize] -= 1;
        }
        count.sort_unstable();
        //println!("{:?}",count);
        let mut sum:i32 = 0;
        for i in 0..26 {
            sum += count[i]*(i/8+1) as i32;
        }

        0-sum
    }
}
```
# 2024-08-05
[2053. Kth Distinct String in an Array](https://leetcode.com/problems/kth-distinct-string-in-an-array/)

Rust makes it really easy.

```Rust
use std::collections::HashMap;

impl Solution {
    pub fn kth_distinct(arr: Vec<String>, k: i32) -> String {
        let mut count = HashMap::new();
        for a in arr.iter() {
            *count.entry(a).or_insert(0) += 1;
        }
        //println!("{:?}",count);
        let mut i = 0;
        let mut out = String::new();
        for a in arr.iter() {
            if count[a] == 1 {
                i += 1;
                if i == k {
                    out = a.clone();
                    break;
                }
            }
        }
        out
    }
}
```
# 2024-08-04
[1508. Range Sum of Sorted Subarray Sums](https://leetcode.com/problems/range-sum-of-sorted-subarray-sums/)

Keeping it simple.

Using if check instead of modulo is faster for each round of the loop. Faster than using i64 adds. Using unstable sort makes it considerably faster.

The top solution is insane. Using a heap is really the solution. (To get out the minimum subarray sum...) Sometimes I wish I figured it out by myself... But I should really learn the tricks from other people.

## Optimized
```Rust
impl Solution {
    pub fn range_sum(nums: Vec<i32>, n: i32, left: i32, right: i32) -> i32 {
        let mut csum = Vec::new();
        let mut sum = 0;

        for e in nums.iter() {
            csum.push(sum);
            sum += e;
        }
        csum.push(sum);

        let mut subs = Vec::new();
        for i in 0..csum.len() {
            for j in i+1..csum.len() {
                subs.push(csum[j]-csum[i]);
            }
        }
        subs.sort_unstable();

        let left = (left-1) as usize;
        let right = right as usize;
        let mut out = 0;
        for i in left..right {
            out += subs[i];
            if out > 1000000007 {
                out -= 1000000007;
            }
        }

        out
    }
}
```

## First
```Rust
impl Solution {
    pub fn range_sum(nums: Vec<i32>, n: i32, left: i32, right: i32) -> i32 {
        let mut csum = Vec::new();
        let mut sum = 0;

        for e in nums.iter() {
            csum.push(sum);
            sum += e;
        }
        csum.push(sum);

        let mut subs = Vec::new();

        for i in 0..csum.len() {
            for j in i+1..csum.len() {
                subs.push(csum[j]-csum[i]);
            }
        }

        subs.sort();
        //println!("{:?}",subs);

        let left = (left-1) as usize;
        let right = right as usize;
        let mut out: i64 = 0;

        for i in left..right {
            out += subs[i] as i64;
        }

        (out%1000000007) as i32
    }
}
```
# 2024-08-03
[1460. Make Two Arrays Equal by Reversing Subarrays](https://leetcode.com/problems/make-two-arrays-equal-by-reversing-subarrays/)

Pretty simple.

It's interesting to think if an array can be transformed into any arbitrary order by reversing subarrays. Intuitively, it seems possible.

This can be done via induction. We can put any element we want at the start of the array by reversing the element to the front. (Any index n can be flipped into index 0.) After the reversing step, the element is in the correct position... Then it becomes part of the "done" subarray. Now we have a new front element and thus we continue the induction step until the unfinished subarray has size 1.

```Rust
impl Solution {
    pub fn can_be_equal(target: Vec<i32>, arr: Vec<i32>) -> bool {
        let mut target = target;
        let mut arr = arr;

        target.sort();
        arr.sort();

        target == arr
    }
}
```
# 2024-08-02
[2134. Minimum Swaps to Group All 1's Together II](https://leetcode.com/problems/minimum-swaps-to-group-all-1s-together-ii/)

I over-complicated things. Took me a while to really figure out the problem... Though the key idea was sort of hovering inside my head from the start. Thought too much into the "swapping" aspect of the problem and tried to prematurely optimize things. Thought I was being smart by making it more "sophisticated"... In the end it's just creating a cumulative sum.

Another key aspect is to simply duplicate the numbers to have it wrap around.

Maybe I was too tired today and got ~~duped~~ "suggested" by the question's description...

Running a for loop twice was easier than I thought...
## Optimized
```Rust
impl Solution {
    pub fn min_swaps(nums: Vec<i32>) -> i32 {
        let total: i32 = nums.iter().sum();
        let total = total as usize;
        
        let mut sum: i32 = nums.iter().take(total).sum();
        let mut max = sum;

        for i in total..nums.len() {
            sum -= nums[i-total];
            sum += nums[i];
            if max < sum {
                max = sum;
            }
        }
        for i in 0..total {
            sum -= nums[i+nums.len()-total];
            sum += nums[i];
            if max < sum {
                max = sum;
            }
        }

        total as i32 - max
    }
}
```
## Solution
```Rust
impl Solution {
    pub fn min_swaps(nums: Vec<i32>) -> i32 {
        let total: i32 = nums.iter().sum();
        let total = total as usize;
        let mut nums = nums.repeat(2);
        
        let mut ccost = Vec::new();
        let mut cost = 0;

        for n in nums.iter() {
            ccost.push(cost);
            if *n==0 {
                cost += 1;
            }
        }
        ccost.push(cost);

        //println!("{:?}",nums);
        //println!("{:?}",ccost);

        let mut min = ccost[total];

        for i in total..ccost.len() {
            let val = ccost[i]-ccost[i-total];
            //print!("{} ",ccost[i]-ccost[i-total]);
            if min > val {
                min = val;
            }
        }

        min
    }
}
```
## Attempt 2
```Rust
impl Solution {
    pub fn min_swaps(nums: Vec<i32>) -> i32 {
        let mut ones = Vec::new();
        let mut zeros = Vec::new();

        let mut wasone = true;
        ones.push(0);

        for n in nums.iter() {
            match n {
                0 => {
                    if wasone {
                        zeros.push(1);
                        wasone = false;
                    } else {
                        *zeros.last_mut().unwrap() += 1;
                    }
                },
                1 => {
                    if wasone {
                        *ones.last_mut().unwrap() += 1;
                    } else {
                        ones.push(1);
                        wasone = true;
                    }
                },
                _ => ()
            }
        }

        if zeros.len()<2 {
            return 0;
        }

        if zeros.len()<ones.len() {
            zeros.push(0);
        }

        println!("zeros: {:?}",zeros);
        println!("ones: {:?}",ones);
        println!("{} {}",zeros.iter().sum::<i32>(),ones.iter().sum::<i32>());

        let mut total = ones.iter().sum::<i32>();

        let mut ccost = Vec::new();
        let mut cost = 0;
        let mut csum = Vec::new();
        let mut sum = 0;

        for i in 0..zeros.len() {
            sum += ones[i];
            csum.push(sum);
            ccost.push(cost);
            cost += zeros[i];
            sum += zeros[i];
        }
        for i in 0..zeros.len() {
            sum += ones[i];
            csum.push(sum);
            ccost.push(cost);
            cost += zeros[i];
            sum += zeros[i];
        }
        println!("ccost: {:?}",ccost);
        println!("csum: {:?}",csum);
        println!("len: {} {}",ccost.len(),csum.len());

        let mut a = 0 as usize;
        let mut b = 1 as usize;

        while a <= zeros.len() {
            while csum[b]-csum[a] < total {
                b += 1;
            }
            println!("({},{}) a:{} b:{}",csum[b]-csum[a],ccost[b]-ccost[a],a,b);
            a += 1;
            while csum[b]-csum[a] > total {
                b -= 1;
                print!(".");
            }
        }


        1000
    }
}
```
## Attempt 1
```Rust
impl Solution {
    pub fn min_swaps(nums: Vec<i32>) -> i32 {
        let mut ones = Vec::new();
        let mut zeros = Vec::new();

        let mut wasone = true;
        ones.push(0);

        for n in nums.iter() {
            match n {
                0 => {
                    if wasone {
                        zeros.push(1);
                        wasone = false;
                    } else {
                        *zeros.last_mut().unwrap() += 1;
                    }
                },
                1 => {
                    if wasone {
                        *ones.last_mut().unwrap() += 1;
                    } else {
                        ones.push(1);
                        wasone = true;
                    }
                },
                _ => ()
            }
        }

        /*
        if ones.len()==zeros.len() && ones[0]==0 && ones.len()>1 {
            ones.rotate_left(1);
            ones.pop();
            zeros[0] += zeros.pop().unwrap();
            zeros.rotate_left(1);
        }

        if ones.len()>zeros.len() && ones.len()>1 {
            ones[0] += ones.pop().unwrap();
        }
        */

        let mut cost = 0;

        while ones.len()>1 && zeros.len()>1 {

            println!("zeros: {:?}",zeros);
            println!("ones : {:?}",ones);

            if ones.len() > zeros.len() {
                ones[0] += ones.pop().unwrap();
            } else if ones.len() < zeros.len() {
                zeros[0] += zeros.pop().unwrap();
            } else {
                let mut mo=0;
                for i in 1..ones.len() {
                    if ones[i] < ones[mo] {
                        mo = i;
                    }
                }
                let mut mz=0;
                for i in 1..zeros.len() {
                    if zeros[i] < zeros[mz] {
                        mz = i;
                    }
                }
                /*
                if ones[mo] == zeros[mz] {
                    cost += ones[mo];
                    
                    let ro = (mo+1)%ones.len();
                    let rz = (mz+1)%zeros.len();
                    let to = ones[ro];
                    let tz = zeros[rz];

                    ones[mo] += ones[ro];
                    zeros[mz] += zeros[rz];
                    
                    ones.remove(ro);
                    zeros.remove(rz);
                } else if ones[mo] < zeros[mz]
                */
                let m = if ones[mo] < zeros[mz] {
                    ones[mo]
                } else {
                    zeros[mz]
                };

                cost += m;
                ones.remove(mo);
                zeros.remove(mz);
                continue;

                //zeros[mo] += m;
                ones[mo] -= m;
                //ones[mz] += m;
                zeros[mz] -= m;
                cost += m;

                let co = if mo == 0 {
                    zeros.len()-1
                } else {
                    mo-1
                };
                let cz = if mz == 0 {
                    zeros.len()-1
                } else {
                    mz-1
                };
                zeros[co] += m;
                ones[co] += m;

                println!("zeros: {:?}",zeros);
                println!("ones : {:?}",ones);
                println!("{}",cost);

                if mo == mz {

                } else if ones[mo] == 0 {
                    let ci = if mo == 0 {
                        zeros.len()-1
                    } else {
                        mo-1
                    };
                    //zeros[mo] += m;
                    //ones[mz] += m;
                    //zeros[ci] += zeros[mo];
                    ones.remove(mo);
                    zeros.remove(mo);
                } else if zeros[mz] == 0 {
                    let ci = if mz == 0 {
                        zeros.len()-1
                    } else {
                        mz-1
                    };
                    //zeros[mo] += m;
                    //ones[mz] += m;
                    //ones[ci] += ones[mz];
                    ones.remove(mz);
                    zeros.remove(mz);
                } else {
                    ones.remove(mo);
                    zeros.remove(mo);
                }

                /*
                let co = if mo == 0 {
                    zeros.len()-1
                } else {
                    mo-1
                };
                let cz = if mz == 0 {
                    zeros.len()-1
                } else {
                    mz-1
                };
                zeros[co] += zeros[mo];
                ones[co] += ones[mz];
                ones.remove(mo);
                zeros.remove(mz);
                */
            }
        }

            println!("zeros: {:?}",zeros);
            println!("ones : {:?}",ones);

        cost
    }
}
```

# 2024-08-01
[2678. Number of Senior Citizens](https://leetcode.com/problems/number-of-senior-citizens/)

A two line solution.

Looking at a faster solution, you could directly index into the string without converting into a vector of u8 first... Rust Strings are so weird...

That being said, not really any faster... There's a lot of measurement variance... But the faster solution is probably faster though.

```Rust
impl Solution {
    pub fn count_seniors(details: Vec<String>) -> i32 {
        let a = details.iter()
            .map(|x|
                String::from_utf8(
                    x.as_bytes().to_vec()[11..13].to_vec()
                ).unwrap()
                .parse::<i32>().unwrap()
            )
            .collect::<Vec<_>>();
 
        //println!("{:?}",a);

        a.iter().filter(|&&x| x>60).count() as i32
    }
}
```
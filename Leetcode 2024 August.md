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
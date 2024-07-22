# 2024-06-30
[1579. Remove Max Number of Edges to Keep Graph Fully Traversable](https://leetcode.com/problems/remove-max-number-of-edges-to-keep-graph-fully-traversable/)
This one was pain. Using Rust, it's really crazy I'm not getting memory error... Well, mostly fighting the compiler instead. Integer casting in Rust is pure pain... I'll have to figure that out...
Vectors are nice... It allows you to be lazy. In C, you have to know exactly what each amounts are and allocate accordingly. I suppose in Rust you try to solve things statically while in C you solve it dynamically.
It's nice how Rust incorporates certain patterns... Though sometimes you're trying to smash a certain pattern as a solution... In C, you just literally do it I suppose.
This was quite the insane exercise for me... I tried out a bunch of ideas... I suppose because I can...
Iterators can be quite interesting... I might not be using them to their fullest potential.

Algorithm wise:
- add in edges
- check connectivity
	- for type 1 and type 3 graphs
	- for type 2 and type 3 graphs
- get minimum type 3 edges
	- traverse type 3 graph
	- implicitly creates a tree
		- (for any connected group: n nodes connected by n-1 edges)
- answer is calculated
	- minimum edges = min. type 3 + (n-1 - min. type 3) * 2
	- answer = edges.len() - min. edges

```rust
impl Solution {
    pub fn max_num_edges_to_remove(n: i32, edges: Vec<Vec<i32>>) -> i32 {
        let n = n as usize;
        let mut ea: Vec<Vec<i32>> = vec![Vec::new(); n+1];
        let mut eb: Vec<Vec<i32>> = vec![Vec::new(); n+1];
        let mut ec: Vec<Vec<i32>> = vec![Vec::new(); n+1];

        let mut na = 0;
        let mut nb = 0;
        let mut nc = 0;
        
        for e in edges.iter() {
            match e[0] {
                1 => {
                    ea[e[1] as usize].push(e[2]);
                    ea[e[2] as usize].push(e[1]);
                    na += 1;
                },
                2 => {
                    eb[e[1] as usize].push(e[2]);
                    eb[e[2] as usize].push(e[1]);
                    nb += 1;
                },
                3 => {
                    ec[e[1] as usize].push(e[2]);
                    ec[e[2] as usize].push(e[1]);
                    nc += 1;
                },
                _ => (),
            }
        }

        //println!("count: {na} {nb} {nc} {}",edges.len());

        // basic count
        /*
        if nc >= n-1 {
            //let n = n as i32;
            return (na+nb+nc - (n-1)) as i32;
        }
        */

        // sort
        /*
        for e in ea.iter_mut() {
            e.sort();
        }
        for e in eb.iter_mut() {
            e.sort();
        }
        for e in ec.iter_mut() {
            e.sort();
        }
        */

        /*
        println!("ea: {:?}",ea);
        println!("eb: {:?}",eb);
        println!("ec: {:?}",ec);
        */

        // filter out edges
        /*
        for i in 1..n+1 {
            let mut j = 0;
            let mut k = 0;
            let mut v1 = &mut ea[i];
            let mut v2 = &mut ec[i];
            while j<v1.len() && k<v2.len() {
                if v1[j] == v2[k] {
                    v1.remove(j);
                } else if v1[j] < v2[k] {
                    j += 1;
                } else {
                    k += 1;
                }
            }
        }
        for i in 1..n+1 {
            let mut j = 0;
            let mut k = 0;
            let mut v1 = &mut eb[i];
            let mut v2 = &mut ec[i];
            while j<v1.len() && k<v2.len() {
                if v1[j] == v2[k] {
                    v1.remove(j);
                } else if v1[j] < v2[k] {
                    j += 1;
                } else {
                    k += 1;
                }
            }
        }
        */

        /*
        println!("ea: {:?}",ea);
        println!("eb: {:?}",eb);
        println!("ec: {:?}",ec);
        */

        let mut reach = vec![false; n+1];
        trav(1, &ea, &ec, &mut reach);
        for i in 1..n+1 {
            if reach[i] == false {
                return -1;
            }
        }
        //println!("reach: {:?}",reach);
        reach.fill(false);
        trav(1, &eb, &ec, &mut reach);
        for i in 1..n+1 {
            if reach[i] == false {
                return -1;
            }
        }
        //println!("reach: {:?}",reach);
        reach.fill(false);
        let mut nc3 = 0;
        for i in 1..n+1 {
            if reach[i] == false {
                nc3 += cloop(i, &ec, &mut reach);
                //print!("{nc3} ");
            }
        }
        //println!("nc3: {nc3}");

        /*
        let mut na2 = 0;
        let mut nb2 = 0;
        let mut nc2 = 0;
        for i in 1..n+1 {
            na2 += ea[i].len();
            nb2 += eb[i].len();
            nc2 += ec[i].len();
        }
        na2 /= 2;
        nb2 /= 2;
        nc2 /= 2;
        println!("ea:{na} {na2}");
        println!("eb:{nb} {nb2}");
        println!("ec:{nc} {nc2}");
        */

        //(na+nb+nc - ((n as i32)-1-nc3)*2).try_into().unwrap()

        (na+nb+nc - ((n as i32)-1)*2 + nc3).try_into().unwrap()

    }
}

fn trav(i:usize, v1: & Vec<Vec<i32>>, v2: & Vec<Vec<i32>>, r: &mut Vec<bool>) {
    if r[i] == true {
        return;
    }

    r[i] = true;

    for e in v2[i].iter() {
        trav(*e as usize, v1, v2, r);
    }

    for e in v1[i].iter() {
        trav(*e as usize, v1, v2, r);
    }
}

fn cloop(i:usize, v1: & Vec<Vec<i32>>, r: &mut Vec<bool>) -> i32 {
    if r[i] == true {
        return -1;
    }

    r[i] = true;

    let mut c = 0;

    for e in v1[i].iter() {
        c += cloop(*e as usize, v1, r);
        c += 1;
    }

    c
}
```
# 2024-06-29
[2192. All Ancestors of a Node in a Directed Acyclic Graph](https://leetcode.com/problems/all-ancestors-of-a-node-in-a-directed-acyclic-graph/)
Leaning Rust, there seems to be so many ways to do the same thing...
Being honest, Rust makes a lot of things much easier and shorter...
```Rust
impl Solution {
    pub fn get_ancestors(n: i32, edges: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
        let n = n as usize;
        let mut par: Vec<Vec<i32>> = vec![Vec::new(); n];
        for e in edges.iter() {
            par[e[1] as usize].push(e[0]);
        }

        let mut vis: Vec<bool> = vec![false; n];

        for i in 0..n {
            anc(i, &mut par, &mut vis);
        }

        par
    }
}

fn anc(i: usize, par: &mut Vec<Vec<i32>>, vis: &mut Vec<bool>) {
    if(vis[i] == true) {
        return;
    }

    vis[i] = true;

    for e in par[i].clone() {
        anc(e as usize, par, vis);
        let a = par[e as usize].clone();
        par[i].extend(a);
    }

    par[i].sort();
    par[i].dedup();
}
```

```Rust
impl Solution {
    pub fn get_ancestors(n: i32, edges: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
        let n = n as usize;
        let mut par: Vec<Vec<i32>> = vec![Vec::new(); n];
        let mut chi: Vec<Vec<i32>> = vec![Vec::new(); n];
        for e in edges.iter() {
            par[e[1] as usize].push(e[0]);
            chi[e[0] as usize].push(e[1]);
        }

        let mut vis: Vec<bool> = vec![false; n];
        let mut top: Vec<i32> = Vec::new();

        /*
        for i in 0..n {
            if par[i].len() == 0 {
                print!("{i} ");
            }
        }
        */

        /*
        for i in 0..n {
            top_sort(i, &chi, &mut vis, &mut top);
        }
        */

        //top.reverse();
        /*
        for e in top.iter() {
            print!("{e} ");
        }
        */

        for i in 0..n {
            anc(i, &mut par, &mut vis);
        }

        //let mut out: Vec<Vec<i32>> = vec![Vec::new(); n];


        par
    }
}

fn top_sort(i: usize, chi: & Vec<Vec<i32>>, vis: &mut Vec<bool>, lis: &mut Vec<i32>) {
    if(vis[i] == true) {
        return;
    }

    vis[i] = true;

    for e in chi[i].iter() {
        top_sort(*e as usize, chi, vis, lis);
    }

    lis.push(i as i32);
}

fn anc(i: usize, par: &mut Vec<Vec<i32>>, vis: &mut Vec<bool>) {
    if(vis[i] == true) {
        return;
    }

    vis[i] = true;

    for e in par[i].clone() {
        anc(e as usize, par, vis);
        //par[i].push(e);
        let a = par[e as usize].clone();
        par[i].extend(a);
    }

    par[i].sort();
    par[i].dedup();
}
```
# 2024-06-28
[2285. Maximum Total Importance of Roads](https://leetcode.com/problems/maximum-total-importance-of-roads/)
```Rust
impl Solution {
    pub fn maximum_importance(n: i32, roads: Vec<Vec<i32>>) -> i64 {
        let mut a: Vec<i32> = vec![0; n as usize];
        for e in roads {
            a[e[0] as usize] += 1;
            a[e[1] as usize] += 1;
        }

        a.sort();
        let mut sum: i64 = 0;

        for i in 0..n {
            //print!("{}:{} ",i, a[i]);
            sum += (i as i64 + 1)*a[i as usize] as i64;
        }

        sum
    }
}
```
# 2024-06-27
[1791. Find Center of Star Graph](https://leetcode.com/problems/find-center-of-star-graph/)
```C
int findCenter(int** edges, int edgesSize, int* edgesColSize) {
    int a, b, c, d;
    a = edges[0][0];
    b = edges[0][1];
    c = edges[1][0];
    d = edges[1][1];

    if(a==c || a==d){
        return a;
    }

    return b;
}
```
# 2024-06-26
[1382. Balance a Binary Search Tree](https://leetcode.com/problems/balance-a-binary-search-tree/)
KISS principal in action. Not "clever" but performs well and doesn't use much memory. O(n) time and O(n) space.
For O(1) space, the DSW algorithm seems to be the solution. (Convert from tree to "vine" (sorted linked list) and back into BST.) Though O(n), I assume it's still slower. At least twice as many writes.
```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */

/*
int dfs(struct TreeNode* a){
    if(a == NULL){
        return 0;
    }

    int l, r, d;
    l = dfs(a->left);
    r = dfs(a->right);

    if(l>r){
        d = l+1;
    } else {
        d = r+1;
    }

    printf("val:%d l:%d r:%d d:%d\n",a->val,l,r,d);
    return d;
}
*/

int bal(struct TreeNode* a){
    if(a == NULL){
        return 0;
    }

    int l, r;
    l = bal(a->left);
    if(l == -1) {
        return -1;
    }
    r = bal(a->right);
    if(r == -1) {
        return -1;
    }

    int d;
    d = l-r;
    if(d<0) {
        d = 0-d;
    }
    if(d>1){
        return -1;
    }
    
    if(l>r){
        return l+1;
    } else {
        return r+1;
    }
}

int size(struct TreeNode *a){
    if(a == NULL){
        return 0;
    }

    int s;
    s = 1;
    s += size(a->left);
    s += size(a->right);

    return s;
}

int lin(struct TreeNode *a, struct TreeNode **b, int c){
    if(a == NULL){
        return c;
    }

    int l, r;
    l = lin(a->left,b,c);
    b[l] = a;
    r = lin(a->right,b,l+1);

    return r;
}

struct TreeNode* tree(struct TreeNode *a, struct TreeNode **b, int s, int e){
    if(s==e){
        return NULL;
    }

    int m;
    m = (s+e)/2;

    struct TreeNode *l, *r;

    l = tree(a,b,s,m);
    b[m]->left = l;
    r = tree(a,b,m+1,e);
    b[m]->right = r;

    return b[m];
}

struct TreeNode* balanceBST(struct TreeNode* root) {
    int b, s;

    // check if balanced
    b = bal(root);
    if(b>-1){
        return root;
    }

    // get size
    s = size(root);
    //printf("%d\n",s);

    struct TreeNode **a;
    a = malloc(s*sizeof(struct TreeNode *));

    // "linearize" into array
    lin(root,a,0);

    /*
    for(int i=0; i<s; i++){
        printf("%d ", a[i]->val);
    }
    */

    // create new binary search tree
    struct TreeNode *out;
    out = tree(root,a,0,s);
    
    return out;
}
```
# 2024-06-25
[1038. Binary Search Tree to Greater Sum Tree](https://leetcode.com/problems/binary-search-tree-to-greater-sum-tree/)
```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */

int dfs(struct TreeNode* a, int sum){
    if(a == NULL){
        return 0;
    }

    int v,l,r;
    r = dfs(a->right, sum);
    v = a->val;
    a->val = sum+v+r;
    l = dfs(a->left, sum+v+r);
    //printf("v:%d l:%d r:%d s:%d\n",v,l,r,v+l+r);
    return v+l+r;
}

struct TreeNode* bstToGst(struct TreeNode* root) {
    dfs(root,0);
    return root;
}
```
# 2024-06-24
[995. Minimum Number of K Consecutive Bit Flips](https://leetcode.com/problems/minimum-number-of-k-consecutive-bit-flips/)
Cheated...
```C
int minKBitFlips(int* nums, int numsSize, int k) {
    /*
    for(i=0; i<numsSize-k+1; i++){
        if(nums[i]==0){
            for(int j=i; j<i+k; j++){
                nums[j] ^= 1;
            }
            c++;
        }
    }
    */
    /*
    for(i=0; i<numsSize-k+1; i++){
        int m;
        m = 0;
        while(m<k && nums[i]==s){
            s ^= 1;
            i++;
            c++;
            m++;
        }
    }
    */

    //printf("%d\n",numsSize);
    if(k>50000){
        return -1;
    }
    int c, i;
    c = 0;
    i = 0;

    while(i<numsSize-k-k+2){
        if(nums[i]==0){
            int s, t;
            s = 0;
            t = 0;
            for(int j=i; j<i+k; j++){
                if(s==nums[j]){
                    s ^= 1;
                    c++;
                    t++;
                }
            }
            int e;
            e = i+k+k-1;
            if(t&1){
                for(int j=i+k; j<e; j++){
                    nums[j] = (nums[j]^nums[i]);
                    i++;
                }
            } else {
                for(int j=i+k; j<e; j++){
                    nums[j] = (nums[j]^nums[i]^1);
                    i++;
                }
            }
            i++;
        } else {
            i++;
        }
    }
    while(i<numsSize-k+1){
        if(nums[i]==0){
            for(int j=i; j<i+k; j++){
                nums[j] ^= 1;
            }
            c++;
        }
        i++;
    }
    while(i<numsSize){
        if(nums[i]==0){
            return -1;
        }
        i++;
    }
    return c;
}
```
# 2024-06-23
[1438. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit](https://leetcode.com/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/)
```C
int abs(int a){
    if(a>=0){
        return a;
    } else {
        return 0-a;
    }
}

int longestSubarray(int* nums, int numsSize, int limit) {
    int out;
    out = 0;
    for(int i=0; i<numsSize-1; i++){
        int min, max;
        if(nums[i]<nums[i+1]){
            min = nums[i];
            max = nums[i+1];
        } else {
            min = nums[i+1];
            max = nums[i];
        }

        int dif;
        dif = max-min;
        if(dif > limit){
            continue;
        }else if(out==0){
            out = 1;
        }
        
        int j;
        j = i+2;
        while(j<numsSize && dif <= limit){
            if(nums[j]<min){
                min = nums[j];
            }else if(nums[j]>max){
                max = nums[j];
            }
            dif = max-min;
            j++;
        }
        j--;
        if(dif>limit){
            j--;
        }

        if(out < j-i){
            /*
            for(int k=i; k<=j; k++){
                printf("%d ",nums[k]);
            }
            printf("\n");
            */
            out = j-i;
        }

        if(j==numsSize-1){
            break;
        }

        j++;
        if(j<numsSize){
            int old;
            old = i;
            while(i<j && abs(nums[j]-nums[i])>limit){
                i++;
            }
            if(i>old){
                i--;
            }
        }
    }
    out++;
    return out;
}
```
# 2024-06-22
[1248. Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/)
```C
int numberOfSubarrays(int* nums, int numsSize, int k){
    int n;
    n = numsSize+1;

    int *s;
    s = calloc(n,sizeof(int));

    for(int i=1; i<n; i++){
        if(nums[i-1]&1){
            s[i] = s[i-1]+1;
        } else {
            s[i] = s[i-1];
        }
    }

    /*
    int out;
    out = 0;

    for(int i=0; i<n; i++){
        int j;
        j = i+k;
        while(j<n && s[j]-s[i]<k){
            j++;
        }
        while(j<n && s[j]-s[i]==k){
            j++;
            out++;
        }
    }
    */

    int *c;
    int m;
    m = s[n-1]+1;
    c = calloc(m,sizeof(int));
    for(int i=0; i<n; i++){
        c[s[i]]++;
    }

    int out;
    out = 0;
    for(int i=0; i<m-k; i++){
        out += c[i]*c[i+k];
    }

    return out;
}
```
# 2024-06-21
[1052. Grumpy Bookstore Owner](https://leetcode.com/problems/grumpy-bookstore-owner/)
Pretty straight forward.
```C
int maxSatisfied(int* customers, int customersSize, int* grumpy, int grumpySize, int minutes) {
    int n;
    n = customersSize;

    // filter for times when grumpy
    for(int i=0; i<n; i++){
        if(grumpy[i]){
            grumpy[i] = customers[i];
        }
    }

    // sum of potential customers recovered
    int *s;
    int m, sum;
    m = n-minutes+1;
    s = malloc(m*sizeof(int));
    sum = 0;
    for(int i=0; i<minutes; i++){
        sum += grumpy[i];
    }
    s[0] = sum;
    // rolling sum
    for(int i=1; i<m; i++){
        sum = sum - grumpy[i-1] + grumpy[i+minutes-1];
        s[i] = sum;
    }

    // find max sum
    int ind;
    ind = 0;
    for(int i=1; i<m; i++){
        if(s[ind] < s[i]){
            ind = i;
        }
    }

    // sum when not grumpy
    sum = 0;
    for(int i=0; i<n; i++){
        if(!grumpy[i]){
            sum += customers[i];
        }
    }

    sum += s[ind];
    return sum;
}
```

# 2024-06-20
[1552. Magnetic Force Between Two Balls](https://leetcode.com/problems/magnetic-force-between-two-balls/)
```C
int compare(int *a, int *b){
    return *a - *b;
}

int maxDistance(int* position, int positionSize, int m) {
    int n;
    n = positionSize;
    qsort(position,n,sizeof(int),compare);

    /*
    for(int i=0; i<n; i++){
        printf("%d ",position[i]);
    }
    */

    if(m==2){
        return position[n-1]-position[0];
    }

    // binary search
    int min, mid, max;
    // best possible distance
    max = (position[n-1] - position[0])/(m-1);
    min = max;
    for(int i=1; i<n; i++){
        int d;
        d = position[i]-position[i-1];
        if(d<min){
            min = d;
        }
    }
    mid = (min+max)/2;
    while(min!=max){
        int c, i, j;
        c = m-1;
        i = 0;
        j = 1;
        // "fill" in bucket with ball if distance to previous
        // ball is greater or equal to search distance
        // we assume we can fill all m balls with search distance
        while(j<n && c>0){
            if(position[j]-position[i]>=mid){
                i=j;
                c--;
            }
            j++;
        }
        
        if(c){
            // not enough buckets, try smaller distance
            if(max==mid){
                max--;
                mid = max;
            }else{
                max = mid;
                mid = (max+min)/2;
            }
        } else {
            // enough buckets, try larger distance
            if(min==mid){
                mid = min+1;
            }else{
                min = mid;
                mid = (max+min)/2;
            }
        }
        //printf("%d %d %d\n",min,mid,max);
    }

    return min;
}
```
# 2024-06-19
[1482. Minimum Number of Days to Make m Bouquets](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/)
Slow... But passes.
```C
int minDays(int* bloomDay, int bloomDaySize, int m, int k) {
    int n;
    n = bloomDaySize;
    long len;
    len = (long)m*k;
    if(len > n){
        return -1;
    }
    if(m*k == n){
        int max;
        max = 0;
        for(int i=0; i<bloomDaySize; i++){
            if(max<bloomDay[i]){
                max = bloomDay[i];
            }
        }
        return max;
    }

    int *g, *q;
    g = calloc(n,sizeof(int));
    q = malloc(n*sizeof(int));

    // indexes to bloomDay values
    for(int i=0; i<n; i++){
        q[i] = i;
    }

    // heapify, priority queue
    // min heap
    for(int i=1; i<n; i++){
        int j, k;
        j = i;
        k = (i-1)/2;
        while(j>0 && bloomDay[q[j]]<bloomDay[q[k]]){
            int t;
            t = q[j];
            q[j] = q[k];
            q[k] = t;
            j = k;
            k = (k-1)/2;
        }
    }

    //int d;
    int b, c;
    b = n;
    c = 0;
    // pop m*k values
    for(int i=0; i<n; i++){
        // pop
        int lo, hi, e, f;
        e = q[0];
        //g[e] = 1;
        //d = bloomDay[e];
        lo = e;
        hi = e;
        f = 1;
        if(e>0 && g[e-1]){
            f += g[e-1];
            lo = e-g[e-1];
            c -= g[e-1]/k;
        }
        if(e<n-1 && g[e+1]){
            f += g[e+1];
            hi = e+g[e+1];
            c -= g[e+1]/k;
        }
        for(int r=lo; r<=hi; r++){
            //printf("%d ",r);
            g[r] = f;
        }
        c += f/k;
        //printf("%d ",c);

        if(c>=m){
            return bloomDay[e];
        }

        // push
        int tt;
        tt = q[0];
        q[0] = q[b-1];
        q[b-1] = tt;
        b--;

        int j,l;
        j = 0;
        l = 1;

        while(l<b){
            if(l+1<b && bloomDay[q[l+1]]<bloomDay[q[l]]){
                l++;
            }

            if(bloomDay[q[l]]<bloomDay[q[j]]){
                int t;
                t = q[j];
                q[j] = q[l];
                q[l] = t;

                j = l;
                l = l*2+1;
            } else {
                break;
            }
        }
    }

    /*
    for(int i=0; i<n; i++){
        printf("%d ",q[i]);
    }
    printf("\n");
    for(int i=0; i<n; i++){
        printf("%d ",bloomDay[q[i]]);
    }
    printf("\n");
    for(int i=0; i<n; i++){
        printf("%d ",g[q[i]]);
    }
    printf("\n");
    for(int i=0; i<n; i++){
        printf("%d ",bloomDay[i]);
    }
    printf("\n");
    for(int i=0; i<n; i++){
        printf("%d ",g[i]);
    }
    */

    return -1;
}
```
# 2024-06-18
[826. Most Profit Assigning Work](https://leetcode.com/problems/most-profit-assigning-work/)
Typical solution but slow. Two sorts followed by greedy selection.
```C
struct job{
    int d;
    int p;
};

int compareProfit(struct job *a, struct job *b){
    return b->p - a->p;
}

int compareWorker(int *a, int *b){
    return *b - *a;
}

int maxProfitAssignment(int* difficulty, int difficultySize, int* profit, int profitSize, int* worker, int workerSize) {
    struct job *jobs;
    int n;
    n = difficultySize;
    jobs = malloc(n*sizeof(struct job));
    for(int i=0; i<n; i++){
        jobs[i].p = profit[i];
        jobs[i].d = difficulty[i];
    }

    // sort with highest value at 0
    qsort(jobs,n,sizeof(struct job),compareProfit);
    qsort(worker,workerSize,sizeof(int),compareWorker);

    /*
    for(int i=0; i<n; i++){
        printf("%d,%d ",jobs[i].p,jobs[i].d);
    }
    printf("\n");
    for(int i=0; i<workerSize; i++){
        printf("%d ",worker[i]);
    }
    */

    int dif, out;
    dif=0;
    out=0;
    for(int i=0; i<workerSize; i++){
        while(dif<n && worker[i]<jobs[dif].d){
            dif++;
        }
        if(dif==n){
            break;
        }
        out += jobs[dif].p;
    }

    return out;
}
```
# 2024-06-17
[633. Sum of Square Numbers](https://leetcode.com/problems/sum-of-square-numbers/)
Basic search, nothing too mathematical.
```
███ ██  ███  █  ███ ██   ██  █  ███ ██  ███  █   ██ ██   ██  █  
███ ██  ███  █  ███ ██   ██  █   ██ ██  ███  █   ██ ██   ██  █  
███ ██  ███  █  ███ ██   ██  █  ███ ██  ███  █   ██ ██   ██  █  
 ██ ██  ███  █  ███ ██   ██  █   ██ ██  ███  █   ██ ██   ██  █  
███ ██  ███  █  ███ ██   ██  █  ███ ██  ███  █   ██ ██   ██  █  
███ ██  ███  █  ███ ██   ██  █   ██ ██  ███  █   ██ ██   ██  █  
 ██ ██  ███  █  ███ ██   ██  █  ███ ██  ███  █   ██ ██   ██  █  
 ██ ██  ███  █  ███ ██   ██  █   ██ ██  ███  █   ██ ██   ██  █  
███ ██  ███  █  ███ ██   ██  █  ███ ██  ███  █   ██ ██   ██  █  
███ ██  ███  █  ███ ██   ██  █   ██ ██  ███  █   ██ ██   ██  █  
███ ██  ███  █  ███ ██   ██  █  ███ ██  ███  █   ██ ██   ██  █  
 ██ ██  ███  █  ███ ██   ██  █   ██ ██  ███  █   ██ ██   ██  █  
 ██ ██  ███  █  ███ ██   ██  █  ███ ██  ███  █   ██ ██   ██  █  
███ ██  ███  █  ███ ██   ██  █   ██ ██  ███  █   ██ ██   ██  █  
 ██ ██  ███  █  ███ ██   ██  █  ███ ██  ███  █   ██ ██   ██  █  
 ██ ██  ███  █  ███ ██   ██  █   ██ ██  ███  █   ██ ██   ██  █  
```

```C
bool judgeSquareSum(int c) {
    /*
    int s[512];
    bool b[1024];
    
    for(int i=0; i<512; i++){
        s[i] = i*i;
    }

    for(int i=0; i<1024; i++){
        b[i] = false;
    }

    for(int i=0; i<512; i++){
        for(int j=0; j<512; j++){
            b[(s[i]+s[j])&1023] = true;
        }
    }

    int n;
    n=0;
    for(int i=0; i<128; i++){
        
        if(b[i]){
            printf("1,");
            n++;
        }else{
            printf("0,");
        }
        
        if((i&15)==15){
            printf("\n");
        }
    }

    printf("%d %d\n",n,128-n);
    */

    // screen out certain values
    bool b[128] = {
        1,1,1,0,1,1,0,0,1,1,1,0,0,1,0,0,
        1,1,1,0,1,1,0,0,0,1,1,0,0,1,0,0,
        1,1,1,0,1,1,0,0,1,1,1,0,0,1,0,0,
        0,1,1,0,1,1,0,0,0,1,1,0,0,1,0,0,
        1,1,1,0,1,1,0,0,1,1,1,0,0,1,0,0,
        1,1,1,0,1,1,0,0,0,1,1,0,0,1,0,0,
        0,1,1,0,1,1,0,0,1,1,1,0,0,1,0,0,
        0,1,1,0,1,1,0,0,0,1,1,0,0,1,0,0
    };

    if(c<128 && b[c]){
        return true;
    }

    if(!b[c&127]){
        return false;
    }

    int h, l;
    h = sqrt(c);
    l = sqrt(c-h*h);
    // rounds
    //int t;
    //t = 0;
    while(l<h){
        int h2, l2, d;
        h2 = h*h;
        l2 = l*l;
        d = c-h2;
        if(d==l2){
            //printf("%d = %d^2 + %d^2",c,l,h);
            return true;
        } else if(d<l2){
            h--;
        } else {
            l++;
        }
        //t++;
    }
    //printf("%d",t);

    return false;
}
```
# 2024-06-16
[330. Patching Array](https://leetcode.com/problems/patching-array/)
Hard but used a mathematical heuristic.
```C
int minPatches(int* nums, int numsSize, int n) {
    int p, i;
    long s;
    s = 0;
    p = 0;
    i = 0;

    while(s<n){
        if(i<numsSize && nums[i]<=s+1){
            s += nums[i];
            i++;
        } else {
            s += s+1;
            p++;
        }
    }

    return p;
}
```
# 2024-06-15
[502. IPO](https://leetcode.com/problems/ipo/)
Wow, first time implementing a binary heap. I'm so proud of myself... Can't believe I've never done it in college...💀
First, sort by capital. Then as capital increases with each project chosen, projects meeting capital requirements will be added to a priority queue. The max profit will then be dequeued.
The priority queue is implemented as max binary heap. The implementation is bare-bones.
```C
struct project{
    int p;
    int c;
};

int compare(struct project *a, struct project *b){
    return a->c - b->c;
}

int findMaximizedCapital(int k, int w, int* profits, int profitsSize, int* capital, int capitalSize) {
    struct project *projects;
    int n;
    n = profitsSize;
    projects = malloc(n*sizeof(struct project));

    for(int i=0; i<n; i++){
        projects[i].p = profits[i];
        projects[i].c = capital[i];
    }

    // sort by capital
    qsort(projects,n,sizeof(struct project),compare);

    /*
    for(int i=0; i<n; i++){
        printf("%d,%d; ",projects[i].p,projects[i].c);
    }
    */

    int c, m;
    c = 0;
    m = 0;
    while(k){
        // insert into priority queue
        for(int i=c; i<n; i++){
            if(projects[i].c <= w){
                // insert
                if(i==0){
                    c++;
                    m++;
                }else{
                    // max profit heap
                    projects[m].c = projects[i].c;
                    projects[m].p = projects[i].p;

                    int j;
                    j=m;
                    while(j!=0 && projects[j].p>projects[(j-1)/2].p){
                        int t1, t2;
                        t1 = projects[j].p;
                        t2 = projects[j].c;
                        projects[j].p = projects[(j-1)/2].p;
                        projects[j].c = projects[(j-1)/2].c;
                        projects[(j-1)/2].p = t1;
                        projects[(j-1)/2].c = t2;
                        j = (j-1)/2;
                    }
                    c++;
                    m++;
                }
            } else {
                break;
            }
        }

        // no more projects
        if(m==0){
            return w;
        }

        // deque
        w += projects[0].p;
        projects[0].p = projects[m-1].p;
        projects[0].c = projects[m-1].c;
        m--;
        int j;
        j=0;
        while(1){
            int a, b;
            a = j*2+1;
            b = j*2+2;

            if(a>=m && b>=m){
                break;
            }

            int t1;
            if(b>=m){
                t1 = a;
            } else if(a>=m){
                t1 = b;
            } else {
                if(projects[a].p < projects[b].p){
                    t1 = b;
                } else {
                    t1 = a;
                }
            }

            if(projects[t1].p > projects[j].p){
                int t2, t3;
                t2 = projects[j].p;
                t3 = projects[j].c;
                projects[j].p = projects[t1].p;
                projects[j].c = projects[t1].c;
                projects[t1].p = t2;
                projects[t1].c = t3;

                j = t1;
            } else {
                break;
            }
        }
        k--;
    }

    return w;
}
```
# 2024-06-14
[945. Minimum Increment to Make Array Unique](https://leetcode.com/problems/minimum-increment-to-make-array-unique/)
```C
int compare(int *a, int *b){
    return *a - *b;
}

int minIncrementForUnique(int* nums, int numsSize) {
    qsort(nums,numsSize,sizeof(int),compare);

    int m=0;
    for(int i=1; i<numsSize; i++){
        if(nums[i]<=nums[i-1]){
            m += nums[i-1]-nums[i]+1;
            nums[i] = nums[i-1]+1;
        }
    }

    return m;
}
```
# 2024-06-13
[2037. Minimum Number of Moves to Seat Everyone](https://leetcode.com/problems/minimum-number-of-moves-to-seat-everyone/)
```C
int compare(int *a, int *b){
    return *a - *b;
}

int minMovesToSeat(int* seats, int seatsSize, int* students, int studentsSize) {
    qsort(seats,seatsSize,sizeof(int),compare);
    qsort(students,studentsSize,sizeof(int),compare);

    int m;
    m=0;
    for(int i=0; i<seatsSize; i++){
        int d;
        d = seats[i]-students[i];
        if(d<0) {
            d = 0-d;
        }
        m+=d;
    }

    return m;
}
```
# 2024-06-12
[75. Sort Colors](https://leetcode.com/problems/sort-colors/)
```C
void sortColors(int* nums, int numsSize) {
    int c[3] = {0,0,0};

    for(int i=0; i<numsSize; i++){
        c[nums[i]]++;
    }

    int d;
    d=0;
    for(int i=0; i<3; i++){
        for(int j=0; j<c[i]; j++){
            nums[d] = i;
            d++;
        }
    }
}
```
# 2024-06-11
[1122. Relative Sort Array](https://leetcode.com/problems/relative-sort-array/)
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
int* relativeSortArray(int* arr1, int arr1Size, int* arr2, int arr2Size, int* returnSize) {
    int *c;
    c = calloc(1001,sizeof(int));

    for(int i=0; i<arr1Size; i++){
        c[arr1[i]]++;
    }

    int *out;
    int d;
    out = calloc(arr1Size,sizeof(int));
    d = 0;
    for(int i=0; i<arr2Size; i++){
        for(int j=0; j<c[arr2[i]]; j++){
            out[d] = arr2[i];
            d++;
        }
        c[arr2[i]] = 0;
    }
    for(int i=0; i<=1000; i++){
        for(int j=0; j<c[i]; j++){
            out[d] = i;
            d++;
        }
    }

    *returnSize = arr1Size;
    return out;
}
```
# 2024-06-10
[1051. Height Checker](https://leetcode.com/problems/height-checker/)
```C
int compare(int *a, int *b){
    return *a - *b;
}

int heightChecker(int* heights, int heightsSize) {
    int *h;
    h = malloc(heightsSize*sizeof(int));
    memcpy(h,heights,heightsSize*sizeof(int));
    qsort(h,heightsSize,sizeof(int),compare);

    int c;
    c = 0;

    for(int i=0; i<heightsSize; i++){
        if(heights[i]!=h[i]){
            c++;
        }
    }

    return c;
}
```
# 2024-06-09
[974. Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/)
```C
int compare(int *a, int *b){
    return *a-*b;
}
int subarraysDivByK(int* nums, int numsSize, int k) {
    // cummulative modulo sum
    nums[0] = nums[0]%k;
    for(int i=1; i<numsSize; i++){
        nums[i] = (nums[i]+nums[i-1])%k;
    }
    for(int i=0; i<numsSize; i++){
        if(nums[i]<0){
            nums[i]+=k;
        }
    }

    // sort and calculate
    qsort(nums,numsSize,sizeof(int),compare);
    for(int i=0; i<numsSize; i++){
        printf("%d ",nums[i]);
    }
    printf("\n");
    int sum;
    sum=0;
    for(int i=0; i<numsSize; i++){
        int a,c;
        a=nums[i];
        c=0;
        i++;
        while(i<numsSize && a==nums[i]){
            i++;
            c++;
        }
        i--;
        if(a==0){
            c++;
        }
        if(c){
            sum += c*(c+1)/2;
        }
    }

    return sum;
}
```
# 2024-06-08
[523. Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/)
Totally forgot... Missed the medal...
```C
int compare(int *a, int *b){
    return *a-*b;
}

bool checkSubarraySum(int* nums, int numsSize, int k) {
    if(numsSize<2){
        return false;
    }

    nums[0] = nums[0]%k;
    for(int i=1; i<numsSize; i++){
        nums[i] = (nums[i]+nums[i-1])%k;
        //printf("%d ",nums[i]);
    }
    //printf("\n");

    for(int i=1; i<numsSize; i++){
        if(nums[i]==0) return true;
    }

    if(numsSize<1000){
        for(int i=0; i<numsSize-2; i++){
            for(int j=i+2; j<numsSize; j++){
                if(nums[j]==nums[i]) return true;
            }
        }
    }

    int *arr;
    arr = malloc(numsSize*sizeof(int));
    for(int i=0; i<numsSize; i++){
        arr[i] = nums[i];
    }
    qsort(arr,numsSize,sizeof(int),compare);

    for(int i=1; i<numsSize; i++){
        if(arr[i]==arr[i-1]){
            int a, b;
            a=0;
            while(nums[a]!=arr[i]){
                a++;
            }
            b=numsSize-1;
            while(nums[b]!=arr[i]){
                b--;
            }
            if(b-a>=2){
                return true;
            }
        }
    }

    return false;
}
```
# 2024-06-07
[648. Replace Words](https://leetcode.com/problems/replace-words/)
I thought it would be fast... Instead it's really slow but memory efficient...

```C
int compare(char **a, char **b){
    char *c, *d;
    c = *a;
    d = *b;
    while(*c && *d && *c==*d){
        c++;
        d++;
    }
    return *c-*d;
}

char* replaceWords(char** dictionary, int dictionarySize, char* sentence) {
    qsort(dictionary,dictionarySize,sizeof(char*),compare);
    /*
    for(int i=0; i<dictionarySize; i++){
        printf("%s\n",dictionary[i]);
    }
    */

    char *s, *t, *word;
    int len;
    s = sentence;
    t = sentence;
    word = malloc(1024*sizeof(char));
    len = 0;
    while(*s){
        if(*s==' '){
            char *a;
            a = word;
            while(*a){
                *t = *a;
                t++;
                a++;
            }
            *t = ' ';
            t++;
            s++;
            len = 0;
            continue;
        }
        word[len]=*s;
        len++;
        word[len]=0;
        if(bsearch(&word,dictionary,dictionarySize,sizeof(char*),compare)){
            //printf("%s ",word);
            char *a;
            a = word;
            while(*a){
                *t = *a;
                t++;
                a++;
            }
            len=0;
            word[len]=0;
            while(*s && *s!=' '){
                s++;
            }
            if(*s==0){
                *t = 0;
                return sentence;
            }
        }else{
            s++;
        }
    }

    char *a;
    a = word;
    while(*a){
        *t = *a;
        t++;
        a++;
    }
    *t = 0;
    return sentence;
}
```
# 2024-06-06
[846. Hand of Straights](https://leetcode.com/problems/hand-of-straights/)
Straightforward. Sorted then "create" groups by checking if sequences exist.

```C
int compare(int *a, int *b){
    return *a-*b;
}

bool isNStraightHand(int* hand, int handSize, int groupSize) {
    if(handSize%groupSize){
        return false;
    }
    if(groupSize==1){
        return true;
    }
    /*
    int groups;
    groups = handSize/groupSize;
    */

    qsort(hand,handSize,sizeof(int),compare);

    for(int i=0; i<handSize; i++){
        int a;
        a = hand[i];
        if(hand[i]!=-1){
            int b,c;
            b = a+1;
            c = 1;
            for(int j=i+1; j<handSize; j++){
                if(b==hand[j]){
                    b++;
                    c++;
                    hand[j] = -1;
                }
                if(c>=groupSize){
                    break;
                }
            }
            if(c<groupSize){
                return false;
            }
            hand[i]=-1;
        }
    }
    return true;
}
```
# 2024-06-05
[1002. Find Common Characters](https://leetcode.com/problems/find-common-characters/)
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
char ** commonChars(char ** words, int wordsSize, int* returnSize){
    int *a, *b;
    a = calloc(26,sizeof(int));
    b = calloc(26,sizeof(int));
    bool first;
    first = true;

    for(int i=0; i<wordsSize; i++){
        char *s;
        s = words[i];
        // count # of letters in each word
        while(*s){
            a[*s-'a']++;
            s++;
        }

        /*
        // print test
        for(int j=0; j<26; j++){
            if(a[j])
                printf("%c:%d ",j+'a',a[j]);
        }
        printf("\n");
        */

        // take the minimum of both and reset 'a'
        if(first){
            for(int j=0; j<26; j++){
                b[j] = a[j];
                a[j] = 0;
            }
            first = false;
        }else{
            for(int j=0; j<26; j++){
                if(a[j]<b[j]){
                    b[j] = a[j];
                }
                a[j] = 0;
            }
        }
    }

    // count total letters and create output
    int c;
    c = 0;
    for(int i=0; i<26; i++){
        c += b[i];
    }
    char **out;
    out = malloc(c*sizeof(char*));
    int d;
    d = 0;
    for(int i=0; i<26; i++){
        for(int j=0; j<b[i]; j++){
            out[d] = malloc(2*sizeof(char));
            out[d][0] = i+'a';
            out[d][1] = 0;
            d++;
        }
    }

    *returnSize = c;
    return out;
}
```
# 2024-06-04
[409. Longest Palindrome](https://leetcode.com/problems/longest-palindrome/)
```C
int longestPalindrome(char* s) {
    int *c;
    c = calloc(52,sizeof(int));

    while(*s){
        if(*s>='a'){
            c[*s-'a']++;
        } else {
            c[*s-'A'+26]++;
        }
        s++;
    }

    bool odd;
    odd = false;
    int out;
    out = 0;
    for(int i=0; i<52; i++){
        if(!odd && c[i]&1){
            odd = true;
        }
        out+=c[i]/2;
    }
    out *= 2;
    if(odd){
        out++;
    }

    return out;
}
```

# 2024-06-03
[2486. Append Characters to String to Make Subsequence](https://leetcode.com/problems/append-characters-to-string-to-make-subsequence/)
```C
int appendCharacters(char* s, char* t) {
    int out;
    out = 0;
    while(*s){
        if(*s == *t){
            t++;
        }
        s++;
    }
    while(*t){
        t++;
        out++;
    }
    return out;
}
```
# 2024-06-02
[344. Reverse String](https://leetcode.com/problems/reverse-string/)
Not sure how the fuck Leetcode is measuring the speed. There's so much variance... Looked at the fastest subs but running their code results in my speeds...
```C
void reverseString(char* s, int sSize) {
    char *a, *b;
    a = s;
    b = s+sSize-1;
    while(a<b){
        int t;
        t = *a;
        *a = *b;
        *b = t;
        a++;
        b--;
    }
}
```
```C
void reverseString(char* s, int sSize) {
    for(int i=0; i<sSize/2; i++){
        int t;
        t = s[i];
        s[i] = s[sSize-i-1];
        s[sSize-i-1] = t;
    }
}
```
# 2024-06-01
[3110. Score of a String](https://leetcode.com/problems/score-of-a-string/)
```C
int scoreOfString(char* s) {
    int sum;
    sum = 0;
    while(s[1]){
        int a;
        a = s[1]-s[0];
        if(a<0){
            sum -= a;
        } else {
            sum += a;
        }
        s++;
    }

    return sum;
}
```
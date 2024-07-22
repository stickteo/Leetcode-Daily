# 2024-04-30
[1915. Number of Wonderful Substrings](https://leetcode.com/problems/number-of-wonderful-substrings/)
Cheated by looking at titles of solutions.
Too puzzle like. Depends entirely on 1 key detail to make it fast.
Not much algorithm stuff is actually explored...
## Frequency Method
```C
long long wonderfulSubstrings(char * word){
    
    int n;
    n = strlen(word);

    /*
    if(n>80000){
        return 0;
    }
    */

    printf("%d ",n);

    int *odd, *b;
    odd = calloc(n,sizeof(int));
    b = calloc(n,sizeof(int));

    int *lut;
    lut = calloc(1024,sizeof(int));
    lut[0] = 1;
    lut[1] = 1; lut[2] = 1; lut[4] = 1; lut[8] = 1; lut[16] = 1;
    lut[32] = 1; lut[64] = 1; lut[128] = 1; lut[256] = 1; lut[512] = 1;
    
    // reformat
    for(int i=0; i<n; i++){
        b[i] = 1<<(word[i]-'a');
    }

    // first
    odd[0] = b[0];
    for(int i=1; i<n; i++){
        odd[i] = odd[i-1]^b[i];
    }

    long long count;
    count = 0;
    /*
    for(int i=0; i<n; i++){
        if(lut[odd[i]]){
            count++;
        }
    }
    */

    long long *freq;
    freq = calloc(1024,sizeof(long long));
    for(int i=0; i<n; i++){
        freq[odd[i]]++;
    }
    /*
    for(int i=0; i<1024; i++){
        if(freq[i])
        printf("%03x %d\n", i, freq[i]);
    }
    */

    /*
    long base;
    base = freq[0];
    for(int i=0; i<10; i++){
        base += freq[1<<i];
        printf("%d ", freq[1<<i]);
    }
    count += n;
    count += freq[0]*(base-freq[0]);
    
    count += freq[0]*(freq[0]-1)/2;
    for(int i=0; i<10; i++){
        count += freq[1<<i]*(freq[1<<i]-1)/2;
    }

    for(int i=0; i<10; i++){
        for(int j=0; j<10; j++){
            if(i!=j){
                count += freq[1<<i]*freq[(1<<i)+(1<<j)];
            }
        }
    }
    */

    count += freq[0];

    for(int i=0; i<10; i++){
        count += freq[1<<i];
    }

    for(int i=0; i<1024; i++){
        count += freq[i]*(freq[i]-1)/2;

        for(int j=0; j<10; j++){
            int a;
            a = i^(1<<j);
            if(i<a){
                count += freq[i]*freq[a];
            }
        }
    }



    /*
    for(int i=0; i<n; i++){
        printf("%03x\n",odd[i]);
    }
    */

    /*
    int *odd2;
    odd2 = calloc(n,sizeof(int));

    for(int i=0; i<n; i++){
        //printf("%d ",i);
        // calculate next
        for(int j=i+1; j<n; j++){
            odd2[j] = odd[j]^b[i];
        }

        for(int j=i+1; j<n; j++){
            if(lut[odd2[j]]){
                count++;
            }
        }

        int *t;
        t = odd;
        odd = odd2;
        odd2 = t;
    }
    */

    return count;
}
```
## Still Too Slow
```C
long long wonderfulSubstrings(char * word){
    
    int n;
    n = strlen(word);

    printf("%d ",n);
    /*
    if(n>10){
        return 0;
    }
    */

    int *odd, *sum, *b;
    odd = calloc(n,sizeof(int));
    sum = calloc(n,sizeof(int));
    b = calloc(n,sizeof(int));
    
    // reformat
    for(int i=0; i<n; i++){
        b[i] = 1<<(word[i]-'a');
    }

    // first
    odd[0] = b[0];
    if(b[0]){
        sum[0]=1;
    }
    for(int i=1; i<n; i++){
        odd[i] = odd[i-1]^b[i];
    }
    for(int i=1; i<n; i++){
        if(odd[i]<odd[i-1]){
            sum[i] = sum[i-1]-1;
        }else{
            sum[i] = sum[i-1]+1;
        }
    }

    /*
    for(int i=0; i<n; i++){
        printf("%x %x %d\n", b[i], odd[i], sum[i]);
    }
    */

    long long count;
    count = 0;
    for(int i=0; i<n; i++){
        if(sum[i] <= 1){
            count++;
        }
    }

    int *odd2;
    odd2 = calloc(n,sizeof(int));

    for(int i=0; i<n; i++){
        printf("%d ",i);
        // calculate next
        for(int j=i+1; j<n; j++){
            odd2[j] = odd[j]^b[i];
        }
        for(int j=i+1; j<n; j++){
            if(odd2[j]<odd[j]){
                sum[j]--;
            }else{
                sum[j]++;
            }
        }
        // count
        for(int j=i+1; j<n; j++){
            if(sum[j] <= 1){
                count++;
            }
        }

        int *t;
        t = odd;
        odd = odd2;
        odd2 = t;
    }



    return count;
}
```
## Simple, Too long
```C
long long wonder(char *w, int s, int e, int odd){
    long long c;
    c = 0;

    //printf("%d %d ",s,e);

    // jihgfedcba
    int a;
    a = (odd&1)+(odd&2)+(odd&4)+(odd&8)+(odd&16)+(odd&32)+(odd&64)+(odd&128)+(odd&256)+(odd&512);
    if(a<2){
        c++;
    }
    /*
    if(odd==0 || odd==1 || odd==2 || odd==4 || odd==8 || odd==16 || odd==32 || odd==64 || odd==128 || odd==256 || odd==512){
        c++;
        //fwrite(&w[s], 1, e-s+1, stdout);
    }
    */

    //printf("\n");

    int odd2;

    if(w[e+1]){
        odd2 = (1<<(w[e+1]-'a'))^odd;
        //printf("%d ",odd2);
        c += wonder(w, s, e+1, odd2);
    }

    if(s+1==e && w[s+1]){
        odd2 = (1<<(w[s]-'a'))^odd;
        //odd2 = (1<<(w[s+1]-'a'))^odd;
        //printf("%d ",odd2);
        c += wonder(w, s+1, e, odd2);
    }

    return c;
}

long long wonderfulSubstrings(char * word){
    return wonder(word, 0, 0, 1<<(word[0]-'a'));
}
```
# 2024-04-29
[2997. Minimum Number of Operations to Make Array XOR Equal to K](https://leetcode.com/problems/minimum-number-of-operations-to-make-array-xor-equal-to-k/)

## Easy
```C
int minOperations(int* nums, int numsSize, int k) {
    int a;
    a = k;
    for(int i=0; i<numsSize; i++){
        a = nums[i]^a;
    }
    //printf("%d\n",a);

    int c;
    c = 0;
    while(a){
        if(a&1){
            c++;
        }
        a = a>>1;
    }

    return c;
}
```
# 2024-04-28
[834. Sum of Distances in Tree](https://leetcode.com/problems/sum-of-distances-in-tree/
14 hours!
There was a trick to it. DFS to calculate distances for parent nodes. Then another DFS to calculate distances upward...
So close yet so far...
The key idea is calculating the distances DFS style...
Making the child node the root node while making its parent its child.
- Distance(child) = Distance(parent) + (n - Count(child)) - (Count(child))
- Distance(parent) includes the Distance(child).
- Rank up the parent branch by (n - Count(child)).
- Rank down the child branch by Count(child).
Too bad I cheated...
## DFS (Cheated)
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

void dfs(int node, int **cons, int *count, int *size, int *dist, int r, int p){
    dist[node] = r;
    size[node] = 1;

    for(int i=0; i<count[node]; i++){
        int a;
        a = cons[node][i];

        if(a != p){
            //dist[node] += r;
            //size[node] += 1;

            dfs(a, cons, count, size, dist, r+1, node);
            dist[node] += dist[a];
            size[node] += size[a];
        }
    }
}

void dfs2(int node, int **cons, int *count, int *size, int *dist, int r, int p, int n){
    if(p!=-1){
        dist[node] = dist[p] + n - size[node]*2;
    }

    for(int i=0; i<count[node]; i++){
        int a;
        a = cons[node][i];

        if(a != p){
            //dist[a] = dist[node] + n - count[a]*2;
            dfs2(a, cons, count, size, dist, r+1, node, n);
        }
    }
}

int* sumOfDistancesInTree(int n, int** edges, int edgesSize, int* edgesColSize, int* returnSize) {
    if(n==1) {
        int *out;
        out = calloc(1,sizeof(int));
        out[0] = 0;
        *returnSize = 1;
        return out;
    }


    int *count;
    count = calloc(n,sizeof(int));
    for(int i=0; i<edgesSize; i++){
        count[edges[i][0]]++;
        count[edges[i][1]]++;
    }

    int **cons;
    cons = malloc(n*sizeof(int*));
    for(int i=0; i<n; i++){
        cons[i] = calloc(count[i],sizeof(int));
    }

    int *count2;
    count2 = calloc(n,sizeof(int));
    for(int i=0; i<edgesSize; i++){
        int a, b;
        a = edges[i][0];
        b = edges[i][1];
        cons[a][count2[a]] = b;
        count2[a]++;
        cons[b][count2[b]] = a;
        count2[b]++;
    }

    int *size, *dist;
    size = calloc(n,sizeof(int));
    dist = calloc(n,sizeof(int));

    dfs(0, cons, count, size, dist, 0, -1);
    dfs2(0, cons, count, size, dist, 0, -1, n);

    /*
    for(int i=0; i<n; i++){
        printf("%d ",size[i]);
    }
    printf("\n");
    for(int i=0; i<n; i++){
        printf("%d ",dist[i]);
    }
    printf("\n");
    */

 
    *returnSize = n;
    return dist;
}
```
## Rank based
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

int* sumOfDistancesInTree(int n, int** edges, int edgesSize, int* edgesColSize, int* returnSize) {
    if(n==1) {
        int *out;
        out = calloc(1,sizeof(int));
        out[0] = 0;
        *returnSize = 1;
        return out;
    }


    int *count;
    count = calloc(n,sizeof(int));
    for(int i=0; i<edgesSize; i++){
        count[edges[i][0]]++;
        count[edges[i][1]]++;
    }

    int **cons;
    cons = malloc(n*sizeof(int*));
    for(int i=0; i<n; i++){
        cons[i] = calloc(count[i],sizeof(int));
    }

    int *count2;
    count2 = calloc(n,sizeof(int));
    for(int i=0; i<edgesSize; i++){
        int a, b;
        a = edges[i][0];
        b = edges[i][1];
        cons[a][count2[a]] = b;
        count2[a]++;
        cons[b][count2[b]] = a;
        count2[b]++;
    }

    /*
    for(int i=0; i<n; i++){
        printf("%d: ",i);
        for(int j=0; j<count[i]; j++){
            printf("%d ",cons[i][j]);
        }
        printf("\n");
    }
    */

    int *sum;
    sum = calloc(n,sizeof(int));

    /*
    for(int i=0; i<n; i++){
        for(int j=0; j<count[i]; j++){
            int a;
            a = cons[i][j];
            dist[i][a] = 1;
            dist[a][i] = 1;
        }
    }
    */

    // construct tree
    
    int **cons2;
    cons2 = malloc(n*sizeof(int*));
    for(int i=0; i<n; i++){
        cons2[i] = calloc(count[i],sizeof(int));
    }

    for(int i=0; i<n; i++){
        for(int j=0; j<count[i]; j++){
            cons2[i][j] = cons[i][j];
        }
    }

    //bool reach;
    //reach = calloc(n,sizeof(bool));

    int *rank;
    rank = calloc(n,sizeof(int));
    for(int i=0; i<n; i++){
        rank[i] = -1;
    }

    // make 0 root
    rank[0] = 0;
    int *curr, *next, *par;
    curr = calloc(n,sizeof(int));
    next = calloc(n,sizeof(int));
    par = calloc(n,sizeof(int));
    int currn, nextn, r;
    
    currn = count[0];
    for(int i=0; i<currn; i++){
        int a;
        a = cons[0][i];
        curr[i] = a;
        par[a] = 0;
        rank[a] = 1;
    }

    r = 2;
    while(currn){
        nextn = 0;
        
        for(int i=0; i<currn; i++){
            int a;
            a = curr[i];
            for(int j=0; j<count[a]; j++){
                int b;
                b = cons[a][j];
                if(b != par[a]){
                    next[nextn] = b;
                    par[b] = a;
                    rank[b] = r;
                    nextn++;
                }
            }
        }

        int *t;
        t = curr;
        curr = next;
        next = t;
        currn = nextn;
        r++;
    }

    /*
    for(int i=0; i<n; i++){
        printf("%d ",par[i]);
    }
    printf("\n\n");
    for(int i=0; i<n; i++){
        printf("%d ",rank[i]);
    }
    */

    int **dist;
    dist = malloc(n*sizeof(int*));
    for(int i=0; i<n; i++){
        dist[i] = calloc(n,sizeof(int));
    }

    for(int i=0; i<n; i++){
        for(int j=0; j<n; j++){
            if(i==j){
                continue;
            }
            int ra, rb;
            int a, b;
            ra = rank[i];
            rb = rank[j];
            a = i;
            b = j;
            
            while(ra<rb){
                b = par[b];
                rb = rank[b];
            }
            while(rb<ra){
                a = par[a];
                ra = rank[a];
            }
            while(a!=b){
                a = par[a];
                b = par[b];
            }

            dist[i][j] = rank[i]+rank[j]-rank[a]-rank[b];
            dist[j][i] = dist[i][j];
        }
    }

    /*
    int *count3;
    count3 = calloc(n,sizeof(int));
    for(int i=0; i<n; i++){
        count3[i] = count2[i];
    }

    int m;
    while(1){
        m = 0;
        for(int i=0; i<n; i++){
            if(count2[i]==1){
                int a;
                a = cons[i][0];
                reach[i] = true;

                count3[i]--;
                count3[a]--;
            } else if(count2[i]>1) {
                m++;
            }
        }
        for(int i=0; i<n; i++){
            if(count3[i]==1){
                int a;
                a = 
            }
        }
        if(m==0) break;
    }
    */

    /*
    for(int i=0; i<n; i++){
        for(int j=0; j<n; j++){
            printf("%d ",dist[i][j]);
        }
        printf("\n");
    }
    */

    *returnSize = n;
    return sum;
}
```
## Still Slow
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

int sumDist(int parent, int child, int rank, int **cons, int *count){
    int sum;
    sum = rank;

    for(int i=0; i<count[child]; i++){
        int a;
        a = cons[child][i];
        if(a != parent){
            //sum += rank;
            sum += sumDist(child,a,rank+1,cons,count);
        }
    }

    return sum;
}

int* sumOfDistancesInTree(int n, int** edges, int edgesSize, int* edgesColSize, int* returnSize) {
    if(n==1) {
        int *out;
        out = calloc(1,sizeof(int));
        out[0] = 0;
        *returnSize = 1;
        return out;
    }


    int *count;
    count = calloc(n,sizeof(int));
    for(int i=0; i<edgesSize; i++){
        count[edges[i][0]]++;
        count[edges[i][1]]++;
    }

    int **cons;
    cons = malloc(n*sizeof(int*));
    for(int i=0; i<n; i++){
        cons[i] = calloc(count[i],sizeof(int));
    }

    int *count2;
    count2 = calloc(n,sizeof(int));
    for(int i=0; i<edgesSize; i++){
        int a, b;
        a = edges[i][0];
        b = edges[i][1];
        cons[a][count2[a]] = b;
        count2[a]++;
        cons[b][count2[b]] = a;
        count2[b]++;
    }
    free(count2);

    /*
    for(int i=0; i<n; i++){
        printf("%d: ",i);
        for(int j=0; j<count[i]; j++){
            printf("%d ",cons[i][j]);
        }
        printf("\n");
    }
    */

    int *sum;
    sum = calloc(n,sizeof(int));
    /*
    for(int i=0; i<n; i++){
        for(int j=0; j<count[i]; j++){
            sum[i] += sumDist(i,cons[i][j],1,cons,count);
        }
    }
    */

    /*
    int *cumPrev, *cumCurr, *cumNext, *dif;
    cumPrev = calloc(n,sizeof(int));
    cumCurr = calloc(n,sizeof(int));
    cumNext = calloc(n,sizeof(int));
    dif = calloc(n,sizeof(int));

    for(int i=0; i<n; i++){
        cumPrev[i] = 1;
        cumCurr[i] = count[i]+1;
        dif[i] = count[i];
        printf("%d ",dif[i]);
    }
    printf("\n");

    int rank;
    rank = 1;

    int loop;
    loop = 1;

    while(loop){
        for(int i=0; i<n; i++){
            sum[i] += dif[i]*rank;
        }
        rank++;

        // cum(n,r+1) = cum(n,r) + sum(cum(m,r)) - sum(cum(m,r-1)) - m_len*cum(n,r-1);
        // next cummulative (size) =
        //   + curr cummulative
        //   + sum all cummulative neighbors (excludes itself)
        //   - sum all previous cummulative neighbors (excludes itself)
        //   - num_neighbors * prev cummulative
        
        for(int i=0; i<n; i++){
            cumNext[i] = cumCurr[i];
            for(int j=0; j<count[i]; j++){
                cumNext[i] += cumCurr[cons[i][j]];
            }
            for(int j=0; j<count[i]; j++){
                cumNext[i] -= cumPrev[cons[i][j]];
            }
            //cumNext[i] -= count[i]*cumPrev[i];
            cumNext[i] -= count[i];
        }

        loop = 0;
        for(int i=0; i<n; i++){
            dif[i] = cumNext[i]-cumCurr[i];
            printf("%d ",dif[i]);
            loop += dif[i];
        }
        printf("\n");

        int *t;
        t = cumPrev;
        cumPrev = cumCurr;
        cumCurr = cumNext;
        cumNext = t;
    }
    */

    int *curr, *next;
    int currn, nextn;
    curr = calloc(n,sizeof(int));
    next = calloc(n,sizeof(int));
    
    bool *reach;
    reach = calloc(n,sizeof(bool));

    int *rank;
    int rankn;
    rank = calloc(n+16,sizeof(int));

    for(int i=0; i<n; i++){
        // setup
        rank[0] = 1;
        rank[1] = count[i];
        rankn = 2;

        for(int j=0; j<n; j++){
            reach[j] = false;
        }

        currn = 0;
        reach[i] = true;
        for(int j=0; j<count[i]; j++){
            int a = cons[i][j];
            curr[currn] = a;
            reach[a] = true;
            currn++;
        }

        // iterate
        while(currn){
            nextn = 0;
            rank[rankn]=0;
            for(int j=0; j<currn; j++){
                int a = curr[j];
                for(int k=0; k<count[a]; k++){
                    int b = cons[a][k];
                    if(!reach[b]){
                        next[nextn] = b;
                        reach[b] = true;
                        nextn++;
                        rank[rankn]++;
                    }
                }
            }
            rankn++;

            currn = nextn;
            int *t;
            t = curr;
            curr = next;
            next = t;
        }

        // calc sum
        //printf("%d: ",i);
        /*
        for(int j=2; j<rankn; j++){
            rank[j] = rank[j]*j;
        }
        */
        for(int j=1; j<rankn; j++){
            sum[i] += rank[j]*j;
        }
        printf("%d,",sum[i]);
    }

    *returnSize = n;
    return sum;
}
```
## Slow
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

int sumDist(int parent, int child, int rank, int **cons, int *count){
    int sum;
    sum = rank;

    for(int i=0; i<count[child]; i++){
        int a;
        a = cons[child][i];
        if(a != parent){
            //sum += rank;
            sum += sumDist(child,a,rank+1,cons,count);
        }
    }

    return sum;
}

int* sumOfDistancesInTree(int n, int** edges, int edgesSize, int* edgesColSize, int* returnSize) {
    int *count;
    count = calloc(n,sizeof(int));
    for(int i=0; i<edgesSize; i++){
        count[edges[i][0]]++;
        count[edges[i][1]]++;
    }

    int **cons;
    cons = malloc(n*sizeof(int*));
    for(int i=0; i<n; i++){
        cons[i] = calloc(count[i],sizeof(int));
    }

    int *count2;
    count2 = calloc(n,sizeof(int));
    for(int i=0; i<edgesSize; i++){
        int a, b;
        a = edges[i][0];
        b = edges[i][1];
        cons[a][count2[a]] = b;
        count2[a]++;
        cons[b][count2[b]] = a;
        count2[b]++;
    }
    free(count2);

    for(int i=0; i<n; i++){
        printf("%d: ",i);
        for(int j=0; j<count[i]; j++){
            printf("%d ",cons[i][j]);
        }
        printf("\n");
    }

    int *sum;
    sum = calloc(n,sizeof(int));
    for(int i=0; i<n; i++){
        for(int j=0; j<count[i]; j++){
            sum[i] += sumDist(i,cons[i][j],1,cons,count);
        }
    }

    *returnSize = n;
    return sum;
}
```

# 2024-04-27
[514. Freedom Trail](https://leetcode.com/problems/freedom-trail/)
2 hours.
Pretty much a brute force approach. Not using a "searching" algorithm.
Puzzle-like.... Just thinking a lot for the approach... No particular algorithm... Figuring basic things like the distance.

Method:
- Bin letters
- Calculate distance between last letter and current letter.
	- Binning means no need to search for matching letters.
- Add to running sum.
- Loop until final letter.
- Return minimum sum of final letter in key.
## Managing Sums
```C
int findRotateSteps(char* ring, char* key) {
    int *count;
    count = calloc(26,sizeof(int));
    int n;
    n = 0;

    // get counts of each letter
    char *a;
    a = ring;
    while(a[0]){
        count[a[0]-'a']++;
        a++;
        n++;
    }

    // prepare array of indeces to each letter
    int **ind;
    ind = calloc(26,sizeof(int*));
    for(int i=0; i<26; i++){
        ind[i] = calloc(count[i],sizeof(int));
    }

    int *temp;
    temp = calloc(26,sizeof(int));
    for(int i=0; i<n; i++){
        int b;
        b = ring[i]-'a';
        ind[b][temp[b]] = i;
        temp[b]++;
    }
    free(temp);

    /*
    for(int i=0; i<26; i++){
        printf("%c ",i+'a');
        for(int j=0; j<count[i]; j++){
            printf("%d ",ind[i][j]);
        }
        printf("\n");
    }
    */

    // handle first letter
    int *sum, *sum2;
    int b;
    sum = calloc(n,sizeof(int));
    for(int i=0; i<n; i++){
        sum[i] = INT_MAX;
    }

    a = key;
    b = a[0]-'a';
    a++;
    for(int i=0; i<count[b]; i++){
        int d;
        d = ind[b][i];
        if(n-d < d) d = n-d;

        sum[ind[b][i]] = d;
    }

    // main algo
    sum2 = calloc(n,sizeof(int));
    int curr, next;
    curr = b;
    while(a[0]){
        printf("%c ",a[0]);
        next = a[0]-'a';
        
        for(int i=0; i<n; i++){
            sum2[i] = INT_MAX;
        }

        for(int i=0; i<count[curr]; i++){
            for(int j=0; j<count[next]; j++){
                int dist;
                dist = abs(ind[curr][i] - ind[next][j]);
                if(n-dist < dist) dist = n-dist;

                int nextSum;
                nextSum = sum[ind[curr][i]] + dist;
                if(nextSum < sum2[ind[next][j]]) {
                    sum2[ind[next][j]] = nextSum;
                }
            }
        }

        int *t;
        t = sum;
        sum = sum2;
        sum2 = t;

        curr = next;
        a++;
    }

    int out;
    out = sum[ind[curr][0]];
    //printf("%c ",curr+'a');
    //printf("%d ",out);
    for(int i=1; i<count[curr]; i++){
        int d;
        d = sum[ind[curr][i]];
        //printf("%d ",d);
        if(d<out){
            out = d;
        }
    }

    return out+strlen(key);
}
```
# 2024-04-26
[1289. Minimum Falling Path Sum II](https://leetcode.com/problems/minimum-falling-path-sum-ii/)
1 hour.
Described as hard but fairly easy if using a "trick".

## Simple
```C
int minFallingPathSum(int** grid, int gridSize, int* gridColSize) {
    int n;
    n = gridSize;

    int *minsum;
    minsum = calloc(n,sizeof(int));

    for(int i=0; i<n; i++){
        minsum[i] = grid[0][i];
        //printf("%d ",minsum[i]);
    }
    //printf("\n");

    for(int i=1; i<n; i++){
        // find min
        int min[2];
        int ind[2];

        if(minsum[0]<minsum[1]){
            min[0] = minsum[0];
            ind[0] = 0;
            min[1] = minsum[1];
            ind[1] = 1;
        } else {
            min[1] = minsum[0];
            ind[1] = 0;
            min[0] = minsum[1];
            ind[0] = 1;
        }
        
        for(int j=2; j<n; j++){
            if(minsum[j]<min[0]){
                min[1] = min[0];
                ind[1] = ind[0];
                min[0] = minsum[j];
                ind[0] = j;
            }
            if(j!=ind[0] && minsum[j]<min[1]){
                min[1] = minsum[j];
                ind[1] = j;
            }
        }

        // calc sums
        for(int j=0; j<n; j++){
            if(j != ind[0]){
                minsum[j] = grid[i][j] + min[0];
            } else {
                minsum[j] = grid[i][j] + min[1];
            }

            //printf("%d ",minsum[j]);
        }
        //printf("\n");
    }

    int out;
    out = minsum[0];
    for(int i=1; i<n; i++){
        //printf("%d ",minsum[i]);
        if(out > minsum[i]) {
            out = minsum[i];
        }
    }
    
    free(minsum);

    return out;
}
```

# 2024-04-25
[2370. Longest Ideal Subsequence](https://leetcode.com/problems/longest-ideal-subsequence/)
17 hours!
Took a while to figure it out... A greedy approach but not much algorithm magic going on. More puzzle like.
## Simple and Elegant
```C
int longestIdealString(char* s, int k) {
    int *best;
    best = calloc(26,sizeof(int));

    int n;
    n = strlen(s);

    int *len;
    len = calloc(n,sizeof(int));

    for(int i=0; i<n; i++){
        int lo, c, hi;
        c = s[i]-'a';
        lo = c-k;
        hi = c+k;
        if(lo<0) lo=0;
        if(hi>25) hi=25;

        int max;
        int sj;
        max=0;

        for(int j=lo; j<=hi; j++){
            if(best[j]>max){
                max = best[j];
            }
        }

        len[i] = max+1;
        if(best[c] < len[i]){
            best[c] = len[i];
        }
    }

    int max=0;;
    //max = sub[0];
    for(int i=0; i<26; i++){
        //printf("%d ",best[i]);
        if(max<best[i]) max = best[i];
    }

    return max;
}
```
## Complicated, doesn't work
```C
int longestIdealString(char* s, int k) {
    int *count;
    count = calloc(26,sizeof(int));

    int n;
    n = 0;

    char *a;
    a = s;
    while(a[0]){
        count[a[0]-'a']++;
        n++;
        a++;
    }

    /*
    for(int i=0; i<26; i++){
        printf("%c: %d\n",i+'a',count[i]);
    }*/

    int *loc;
    loc = calloc(n,sizeof(int));
    int **letters;
    letters = malloc(26*sizeof(int*));

    int *seq;
    seq = calloc(n,sizeof(int));

    int **seqs;
    seqs = malloc(26*sizeof(int*));

    int *count2;
    count2 = calloc(26,sizeof(int));
    int b;
    b = 0;
    for(int i=0; i<26; i++){
        letters[i] = &loc[b];
        seqs[i] = &seq[b];
        b += count[i];
    }

    for(int i=0; i<n; i++){
        int c;
        c = s[i]-'a';
        letters[c][count2[c]] = i;
        count2[c]++;
    }
    free(count2);

    /*
    for(int i=0; i<26; i++){
        printf("%c: ", i+'a');
        for(int j=0; j<count[i]; j++){
            printf("%d ", letters[i][j]);
        }
        printf("\n");
    }
    */

    int *count3;
    count3 = calloc(26,sizeof(int));

    for(int i=0; i<n; i++){
        int c;
        c = s[i]-'a';

        int lo, hi;
        lo = c-k;
        if(lo < 0) lo = 0;
        hi = c+k;
        if(hi > 25) hi = 25;

        for(int k=lo; k<=hi; k++){
            int j;
            j = count3[k];
            while(j<count[k] && letters[k][j]<i) {
                count3[k]++;
                j++;
            }
            while(j<count[k]) {
                seqs[k][j]++;
                j++;
            }
        }
    }

    for(int i=0; i<26; i++){
        printf("%c: ", i+'a');
        for(int j=0; j<count[i]; j++){
            printf("%d ", seqs[i][j]);
        }
        printf("\n");
    }

    return 0;
}
```
## Naive (Too Slow)
```C
int longestIdealString(char* s, int k) {
    /*
    int max, curr;
    max = 1;
    curr = 1;

    char *c;
    c = s;
    while(c[0]) {
        int dist;
        dist = abs(c[0]-c[1]);
        if(dist<=k){
            curr++;
            if(curr>max) {
                max=curr;
            }
        }else{
            curr=1;
        }
        c++;
    }

    return max;
    */

    int *longest;
    int len;
    len = strlen(s);
    longest = calloc(len,sizeof(int));

    printf("%d\n",len);
    
    for(int i=0; i<len; i++){
        longest[i] = 1;
    }

    for(int i=0; i<len; i++){
        int lo, hi, max;
        lo = s[i]-k;
        hi = s[i]+k;
        max = 0;
        for(int j=0; j<i; j++){
            if(s[j]>=lo && s[j]<=hi && longest[j]>max){
                max = longest[j];
            }
        }
        longest[i] = max+1;
    }

    int out;
    out = 0;
    for(int i=0; i<len; i++){
        if(out < longest[i]){
            out = longest[i];
        }
    }

    return out;
}
```

# 2024-04-24
[1137. N-th Tribonacci Number](https://leetcode.com/problems/n-th-tribonacci-number/)

## Simple
```C
int tribonacci(int n){
    if(n==0) {
        return 0;
    } else if(n==1) {
        return 1;
    } else if(n==2) {
        return 1;
    }

    int m;
    int a[] = {0,1,1};
    m = 2;
    while(m<n){
        int t;
        t = a[2];
        a[2] = a[0]+a[1]+a[2];
        a[0] = a[1];
        a[1] = t;
        m++;
    }

    return a[2];
}
```
# 2024-04-23
[310. Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/)
3 hours. (+A lot of thinking time.)

The trick is to "prune" the leaves off until 1 or 2 nodes are left. The solution can be proven via induction...
## Pruning
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
int* findMinHeightTrees(int n, int** edges, int edgesSize, int* edgesColSize, int* returnSize) {
    // edge cases
    if(n==1){
        int *out;
        out  = calloc(1, sizeof(int));
        *returnSize = 1;
        return out;
    }


    int *lsize;
    lsize = calloc(n, sizeof(int));

    // count connections for each node
    for(int i=0; i<edgesSize; i++){
        lsize[edges[i][0]]++;
        lsize[edges[i][1]]++;
    }

    int **links;
    links = malloc(n*sizeof(int*));

    for(int i=0; i<n; i++){
        links[i] = calloc(lsize[i],sizeof(int));
    }

    int *c;
    c = calloc(n, sizeof(int));

    // record links
    for(int i=0; i<edgesSize; i++){
        int a,b;
        a = edges[i][0];
        b = edges[i][1];
        
        links[a][c[a]] = b;
        c[a]++;
        
        links[b][c[b]] = a;
        c[b]++;
    }

    /*
    for(int i=0; i<n; i++){
        printf("%d(%d): ",i,lsize[i]);
        for(int j=0; j<lsize[i]; j++){
            printf("%d ",links[i][j]);
        }
        printf("\n");
    }
    */

    // prune
    bool done;
    done = false;
    int *list;
    int *list2;
    int listn;
    list = calloc(n, sizeof(int));
    list2 = calloc(n, sizeof(int));
    while(!done){
        // find leaves
        listn = 0;
        done = true;
        for(int i=0; i<n; i++){
            if(lsize[i]==1){
                list[listn] = i;
                listn++;
            } else if(lsize[i]>1) {
                done = false;
            }
        }
        if(done) break;

        // prune
        done = false;
        for(int i=0; i<listn; i++){
            int a;
            a = list[i];

            //printf("%d ",a);

            for(int j=0; j<lsize[a]; j++){
                int b;
                b = links[a][j];

                for(int k=0; k<lsize[b]; k++){
                    if(links[b][k] == a){
                        // search & delete link
                        while(k<lsize[b]-1){
                            links[b][k] = links[b][k+1];
                            k++;
                        }
                        lsize[b]--;

                        if(lsize[b]==0){
                            int *out;
                            out = calloc(1, sizeof(int));
                            out[0] = b;
                            *returnSize = 1;

                            return out;
                        }
                    }
                }
            }
            lsize[a] = 0;
        }

    }

    /*
    for(int i=0; i<listn; i++){
        printf("%d ",list[i]);
    }
    */
    *returnSize = listn;
    return list;
}
```
# 2024-04-22
[752. Open the Lock](https://leetcode.com/problems/open-the-lock/)
4 hours. (+4 hours "thinking".)
## "Naive" approach.
```C

int openLock(char** deadends, int deadendsSize, char* target) {
    int *d;
    d = calloc(deadendsSize,sizeof(int));
    for(int i=0; i<deadendsSize; i++){
        d[i] = (deadends[i][0]-'0')*1000 + (deadends[i][1]-'0')*100 + (deadends[i][2]-'0')*10 + (deadends[i][3]-'0');
    }

    for(int i=0; i<deadendsSize; i++){
        if(d[i] == 0) {
            free(d);
            return -1;
        }
    }

    int tar;
    tar = (target[0]-'0')*1000 + (target[1]-'0')*100 + (target[2]-'0')*10 + (target[3]-'0');

    if(tar==0) {
        free(d);
        return 0;
    }

    int turns;
    int *curr;
    int *next;
    int n;
    bool *exp;
    exp = calloc(10000,sizeof(bool));
    curr = calloc(10000,sizeof(int));
    next = calloc(10000,sizeof(int));
    
    curr[0] = 0;
    exp[0] = true;
    for(int i=0; i<deadendsSize; i++){
        exp[d[i]] = true;
    }

    n = 1;
    turns = 1;
    while(n){
        int m;
        m = 0;
        for(int i=0; i<n; i++){
            int val, a, b, c, d, e;
            val = curr[i];

            //printf("%d ",val);

            a = val/1000;
            b = (val/100)%10;
            c = (val/10)%10;
            d = val%10;

            e = ((a+1)%10)*1000 + b*100 + c*10 + d;
            if(!exp[e]){
                if(e == tar) return turns;
                exp[e] = true;
                next[m] = e;
                m++;
            }

            e = ((a+9)%10)*1000 + b*100 + c*10 + d;
            if(!exp[e]){
                if(e == tar) return turns;
                exp[e] = true;
                next[m] = e;
                m++;
            }

            e = a*1000 + ((b+1)%10)*100 + c*10 + d;
            if(!exp[e]){
                if(e == tar) return turns;
                exp[e] = true;
                next[m] = e;
                m++;
            }

            e = a*1000 + ((b+9)%10)*100 + c*10 + d;
            if(!exp[e]){
                if(e == tar) return turns;
                exp[e] = true;
                next[m] = e;
                m++;
            }

            e = a*1000 + b*100 + ((c+1)%10)*10 + d;
            if(!exp[e]){
                if(e == tar) return turns;
                exp[e] = true;
                next[m] = e;
                m++;
            }

            e = a*1000 + b*100 + ((c+9)%10)*10 + d;
            if(!exp[e]){
                if(e == tar) return turns;
                exp[e] = true;
                next[m] = e;
                m++;
            }

            e = a*1000 + b*100 + c*10 + ((d+1)%10);
            if(!exp[e]){
                if(e == tar) return turns;
                exp[e] = true;
                next[m] = e;
                m++;
            }

            e = a*1000 + b*100 + c*10 + ((d+9)%10);
            if(!exp[e]){
                if(e == tar) return turns;
                exp[e] = true;
                next[m] = e;
                m++;
            }
        }
        int *temp;
        temp = curr;
        curr = next;
        next = temp;
        n = m;

        turns++;
    }

    printf("%d\n",turns);

    return -1;
}
```
# 2024-04-21
[1971. Find if Path Exists in Graph](https://leetcode.com/problems/find-if-path-exists-in-graph/)
4 hours.
## Hybrid (hard to find singular solution, spent too much time)
```C

bool dfs(int** e, int es, bool* exp, int s, int d) {
    // dfs
    for(int i=0; i<es; i++){
        if(!exp[i]){
            if(e[i][0] == s || e[i][1] == s){
                int other;
                other = e[i][0]+e[i][1]-s;
                if(other == d) return true;

                exp[i] = 1;
                if(dfs(e,es,exp,other,d)){
                    return true;
                }
            }
        }
    }

    return false;
}

bool validPath(int n, int** edges, int edgesSize, int* edgesColSize, int source, int destination) {
    if(source==destination) return true;

    if(n<500){
        bool *exp;
        exp = calloc(edgesSize,sizeof(bool));
        return dfs(edges, edgesSize, exp, source, destination);
    }

    int *groups;
    int *merge;
    int *lead;
    int g;
    groups = calloc(n, sizeof(int));
    merge = calloc(n, sizeof(int));
    lead = calloc(n, sizeof(int));

    for(int i=0; i<n; i++){
        lead[i] = -1;
    }

    g = 1;

    for(int i=0; i<edgesSize; i++){
        int a, b;
        a = edges[i][0];
        b = edges[i][1];
        
        // add new group
        if(groups[a]==0 && groups[b]==0) {
            groups[a]=g;
            groups[b]=g;
            lead[g]=a;
            g++;
        } else if(groups[a]) {
            // add to group
            if(groups[b]==0){
                groups[b]=groups[a];
            } else {
                if(groups[a]<groups[b]){
                    merge[b] = a;
                } else {
                    merge[a] = b;
                }
            }
        } else {
            groups[a] = groups[b];
        }
    }

    if(groups[source] == groups[destination]) {
        free(groups);
        free(merge);
        return true;
    }
    
    /*
    for(int i=0; i<n; i++){
        printf("%d ",groups[i]);
    };
    printf("\n");

    for(int i=0; i<n; i++){
        printf("%d ",merge[i]);
    };
    printf("\n");
    */

    int ms, gs;
    gs = groups[source];
    ms = merge[source];
    while(ms){
        gs = groups[ms];
        ms = merge[ms];
    }

    int md, gd;
    gd = groups[destination];
    md = merge[destination];
    while(md){
        gd = groups[md];
        md = merge[md];
    }

    //printf("%d %d\n",gs,gd);

    if(gs==gd) {
        free(groups);
        free(merge);
        return true;
    }

    // merge groups
    for(int i=0; i<n; i++){
        if(merge[i]){
            int a, b;
            a = groups[i];
            b = groups[merge[i]];

            for(int j=0; j<n; j++){
                if(groups[j]==a){
                    groups[j] = b;
                }
            }
        }
    }

    bool out;
    out = groups[source] == groups[destination];

    free(groups);
    free(merge);

    return out;
}
```
## Groups I (sometimes too slow)
```C
bool validPath(int n, int** edges, int edgesSize, int* edgesColSize, int source, int destination) {
    
    int *groups;
    int *merge;
    int g;
    groups = calloc(n, sizeof(int));
    merge = calloc(n, sizeof(int));
    g = 1;

    for(int i=0; i<edgesSize; i++){
        int a, b;
        a = edges[i][0];
        b = edges[i][1];
        
        // add new group
        if(groups[a]==0 && groups[b]==0) {
            groups[a]=g;
            groups[b]=g;
            g++;
        } else if(groups[a]) {
            // add to group
            if(groups[b]==0){
                groups[b]=groups[a];
            } else {
                // merge groups
                /*
                int last;
                last = groups[b];
                for(int j=0; j<n; j++){
                    if(groups[j] == last){
                        groups[j] = groups[a];
                    }
                }
                */
                //merge[b] = groups[a];
                if(groups[a]<groups[b]){
                    merge[b] = a;
                } else {
                    merge[a] = b;
                }
            }
        } else {
            groups[a] = groups[b];
        }
    }

    //bool out;
    if(groups[source] == groups[destination]) {
        free(groups);
        free(merge);
        return true;
    }
    
    /*
    for(int i=0; i<n; i++){
        printf("%d ",groups[i]);
    };
    printf("\n");

    for(int i=0; i<n; i++){
        printf("%d ",merge[i]);
    };
    printf("\n");
    */

    int ms, gs;
    gs = groups[source];
    ms = merge[source];
    while(ms){
        gs = groups[ms];
        ms = merge[ms];
    }

    int md, gd;
    gd = groups[destination];
    md = merge[destination];
    while(md){
        gd = groups[md];
        md = merge[md];
    }

    //printf("%d %d\n",gs,gd);

    if(gs==gd) {
        free(groups);
        free(merge);
        return true;
    }

    // merge groups
    for(int i=0; i<n; i++){
        if(merge[i]){
            int a, b;
            a = groups[i];
            b = groups[merge[i]];

            for(int j=0; j<n; j++){
                if(groups[j]==a){
                    groups[j] = b;
                }
            }
        }
    }

    bool out;
    out = groups[source] == groups[destination];

    free(groups);
    free(merge);

    return out;
}
```

## DFS (too slow)
```C
bool validPath2(int** e, int es, bool* exp, int s, int d) {
    // dfs
    for(int i=0; i<es; i++){
        if(!exp[i]){
            if(e[i][0] == s || e[i][1] == s){
                int other;
                other = e[i][0]+e[i][1]-s;
                /*
                if(e[i][0] == d || e[i][1] == d){
                    return true;
                }
                */
                if(other == d) return true;

                exp[i] = 1;
                //int source2;
                /*
                if(e[i][0] == s){
                    source2 = e[i][1];
                } else {
                    source2 = e[i][0];
                }
                */
                //source2 = e[i][0]+e[i][1]-s;
                if(validPath2(e,es,exp,other,d)){
                    return true;
                }
            }

            if(validPath2(e,es,exp,source,edges[i][0])){
                bool *exp2;
                exp2 = calloc(edgesSize,sizeof(bool));
                int source2 = edges[i][0];
                if(!validPath2(edges,edgesSize,exp2,source2,destination)){
                    free(exp);
                    exp = exp2;
                } else {
                    printf("reach2");
                    free(exp2);
                }
            }
        }
    }

    return false;
}

bool validPath(int n, int** edges, int edgesSize, int* edgesColSize, int source, int destination) {
    
/*
    int **e;
    e = malloc(sizeof(int*)*n);

    int *a;
    a = malloc(sizeof(int)*2*n);
    for(int i=0; i<n; i++){
        a[i*2] = edges[i][0];
        a[i*2+1] = edges[i][1];
        e[i] = &a[i*2];
    }
*/
    // edge cases
    if(source==destination) return true;

    bool *exp;
    exp = calloc(edgesSize,sizeof(bool));

    //depth first search
    for(int i=0; i<edgesSize; i++){
        if(!exp[i]){
            if(edges[i][0] == source || edges[i][1] == source){
                int other;
                other = edges[i][0] + edges[i][1] - source;
                /*
                if(edges[i][0] == destination || edges[i][1] == destination){
                    return true;
                }
                */
                if(other == destination) return true;

                // other is not destination
                // check if other reaches destination
                exp[i] = 1;
                //int source2;
                /*
                if(edges[i][0] == source){
                    source2 = edges[i][1];
                } else {
                    source2 = edges[i][0];
                }
                */
                //source2 = edges[i][0]+edges[i][1]-source;
                if(validPath2(edges,edgesSize,exp,other,destination)){
                    return true;
                }
            }

            // both edges are not source
            // check if source reaches those nodes
            if(validPath2(edges,edgesSize,exp,source,edges[i][0])){
                bool *exp2;
                exp2 = calloc(edgesSize,sizeof(bool));
                int source2 = edges[i][0];
                if(!validPath2(edges,edgesSize,exp2,source2,destination)){
                    free(exp);
                    exp = exp2;
                } else {
                    printf("reach2");
                    free(exp2);
                }
            }

            if(validPath2(edges,edgesSize,exp,source,edges[i][1])){
                bool *exp2;
                exp2 = calloc(edgesSize,sizeof(bool));
                int source2 = edges[i][1];
                if(!validPath2(edges,edgesSize,exp2,source2,destination)){
                    free(exp);
                    exp = exp2;
                } else {
                    printf("reach2");
                    free(exp2);
                }
            }
        }
    }


    free(exp);

    return false;
}
```

# 2024-04-20
1992 Find All Groups of Farmland
1 hour.
## First
```C
/**
 * Return an array of arrays of size *returnSize.
 * The sizes of the arrays are returned as *returnColumnSizes array.
 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
 */

struct group {
    int r1;
    int c1;
    int r2;
    int c2;
    struct group *next;
};

int** findFarmland(int** land, int landSize, int* landColSize, int* returnSize, int** returnColumnSizes) {
    //for(int i=0; i<)
    //int* a = malloc(sizeof(int)*4);
    //a[0]=0; a[1]=0; a[2]=1; a[3]=1;
    //int** b = malloc(sizeof(int*));
    //b[0] = a;

    struct group *head;
    head = malloc(sizeof(struct group));
    struct group *curr;
    curr = head;

    int n;
    n = 0;

    //printf("landSize = %d\n",landSize);
    for(int i=0; i<landSize; i++){
        //printf("landColSize[%d] = %d\n", i, landColSize[i]);
        for(int j=0; j<landColSize[i]; j++){
            //printf("%d ", land[i][j]);
            if(land[i][j]==1) {
                // find limits of rectangle
                int k = i+1;
                int l = j+1;
                while(k<landSize && land[k][j]==1) k++;
                while(l<landColSize[i] && land[i][l]==1) l++;
                k--;
                l--;
                //printf("%d %d %d %d\n",i,j,k,l);

                // set rectangle to zero
                for(int i2=i; i2<=k; i2++){
                    for(int j2=j; j2<=l; j2++){
                        land[i2][j2] = 0;
                    }
                }

                curr->r1 = i;
                curr->c1 = j;
                curr->r2 = k;
                curr->c2 = l;
                curr->next = malloc(sizeof(struct group));

                curr = curr->next;
                curr->next = NULL;

                n++;
            }
        }
    }

    int **out = malloc(sizeof(int*) * n);
    int *c = malloc(sizeof(int) * n);
    
    struct group *g;
    g = head;
    //while(g != curr){
    //    printf("%d %d %d %d\n", g->r1, g->c1, g->r2, g->c2);
    //    g = g->next;
    //}
    for(int i=0; i<n; i++){
        out[i] = g;
        c[i] = 4;
        g = g->next;
    }

    *returnSize = n;
    //int* c = malloc(sizeof(int));
    //c[0] = 4;
    *returnColumnSizes = c;

    return out;
}
```

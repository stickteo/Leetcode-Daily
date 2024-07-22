# 2024-05-31
[260. Single Number III](https://leetcode.com/problems/single-number-iii/)
I wanted to check how slow it would be to use sort...
Just use enough memory to make the hash work...
Cheated and read best solution... Involves using XOR.
## Okay, just use a lot of memory
```C
// uint hash(uint k){
//     uint a;
//     a = k*k+11;
//     return ((a>>22)^(a>>11)^a)&2047;
// }
uint hash(uint k){
    //uint a;
    //a = k*k+11;
    //a = k;
    return ((k>>28)^(k>>14)^k)&16383;
}

int* singleNumber(int* nums, int numsSize, int* returnSize) {
    if(numsSize==2){
        *returnSize = 2;
        return nums;
    }

    int load, keys;

    int *arr, *arr2, *sta, *sta2;
    arr = calloc(16384,sizeof(int));
    //arr2 = calloc(16384,sizeof(int));
    sta = calloc(16384,sizeof(int));
    //sta2 = calloc(16384,sizeof(int));
    load = 0;
    keys = 0;

    for(int i=0; i<numsSize; i++){
        uint h;
        h = hash(nums[i]);

        if(sta[h]==0){
            // insert
            arr[h] = nums[i];
            sta[h] = 1; // occuppied
            //load++;
            keys++;
        } else {
            while(sta[h]!=0){
                if(sta[h]==1 && nums[i]==arr[h]){
                    break;
                }
                h = (h+1)&16383;
            }
            if(sta[h]==1){
                // delete
                //arr[h] = 0;
                sta[h] = 2;
                load++;
                keys--;
            }else{
                // insert
                arr[h] = nums[i];
                sta[h] = 1;
                keys++;
            }
        }

        /*
        if(load>=4096){
            // recalculate hash table
            printf(".");
            memset(sta2,0,sizeof(int)*16384);
            for(int j=0; j<16384; j++){
                if(sta[j]==1){
                    uint h;
                    h = hash(arr[j]);
                    arr2[h] = arr[j];
                    sta2[h] = 1;
                }
            }

            int *t;
            t = arr;
            arr = arr2;
            arr2 = t;
            t = sta;
            sta = sta2;
            sta2 = t;

            load = 0;
        }
        */
    }


    int *out;
    out = calloc(16,sizeof(int));
    int c;
    c = 0;
    printf("load: %d, keys: %d\n",load,keys);
    for(int i=0; i<16383; i++){
        if(sta[i]==1){
            printf("%d ", arr[i]);
            out[c] = arr[i];
            c++;
        }
    }


    *returnSize = 2;
    return out;
}
```

## Hash Attempt
```C
int* singleNumber(int* nums, int numsSize, int* returnSize) {
    if(numsSize==2){
        *returnSize = 2;
        return nums;
    }

    int *arr;
    int *sta;
    arr = calloc(2048,sizeof(int));
    sta = calloc(2048,sizeof(int));

    for(int i=0; i<numsSize; i++){
        // hash
        uint h, a;
        //h = ((nums[i]>>22)^(nums[i]>>11)^nums[i])&2047;
        a = (uint)(nums[i]+11)*nums[i];
        h = ((a>>22)^(a>>11)^a)&2047;


        if(sta[h]==0){
            // insert
            arr[h] = nums[i];
            sta[h] = 1; // occuppied
        } else if (sta[h]==1) {
            if(arr[h] == nums[i]){
                int j, h2;
                j = h;
                sta[h] = 0;
                // find next
                do {
                    j = (j+1)&2047;
                }
                while(sta[j]==1);

                do {
                    j = (j-1)&2047;
                    uint b;
                    b = (uint)(arr[i]+11)*arr[i];
                    h2 = ((b>>22)^(b>>11)^b)&2047;
                } while(j==h2 && sta[j]==1);

                if(j<h){
                    j+=2048;
                }
                if(j<h+16){
                    arr[h] = arr[j&2047];
                    sta[h] = 1;
                    sta[j&2047] = 0;
                } else {
                    sta[h] = 0;
                }

                continue;
            }
            // insert
            while(sta[h]!=1){
                h = (h+1)&2047;
            }
            arr[h] = nums[i];
            sta[h] = 1;
        } else if (sta[h]==2) {
            // insert
            arr[h] = nums[i];
            sta[h] = 1;
        }
    }


    int *out;
    out = calloc(16,sizeof(int));
    int c;
    c = 0;
    for(int i=0; i<2048; i++){
        if(sta[i]==1){
            printf("%d ", arr[i]);
            out[c] = arr[i];
            c++;
        }
    }


    *returnSize = 2;
    return out;
}
```
## Cheating by using sort
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

int compare(const int *a, const int *b){
    if(*a<*b){
        return -1;
    }else if(*a==*b){
        return 0;
    }else{
        return 1;
    }
}

int* singleNumber(int* nums, int numsSize, int* returnSize) {
    if(numsSize==2){
        *returnSize = 2;
        return nums;
    }
    /*
    printf("%x %x\n",&nums,&returnSize);
    printf("%x %x\n",nums,returnSize);
    */

    qsort(nums,numsSize,sizeof(int),compare);
    int *out;
    out = calloc(2,sizeof(int));
    int j=0;
    for(int i=0; i<numsSize-1; i++){
        if(nums[i]==nums[i+1]){
            i++;
        }else{
            out[j] = nums[i];
            j++;
            if(j>=2){
                break;
            }
        }
    }
    if(j<2){
        out[j] = nums[numsSize-1];
    }

    *returnSize = 2;
    return out;
}
```
# 2024-05-30
[1442. Count Triplets That Can Form Two Arrays of Equal XOR](https://leetcode.com/problems/count-triplets-that-can-form-two-arrays-of-equal-xor/)
The visual example demonstrates how the first solution gets optimized into the second solution. This transforms the problem from N^3 to N^2. From counting singles to multiples at once.
## Visual Example
```
  2 3 1 6 7 (original)
0 2 1 0 6 1 (cummulative xor)
* *   +     1
*   * +     1
*     *     0
  * *       0
    * *   + 1
    *   * + 1
      * *   0
(1+1)+(1+1) = 4
(2)+(2) = 4

  1 1 1 1 1 (original)
0 1 0 1 0 1 (cumm xor)
* * +   +   2
*   *   +   1
*     * +   1
*       *   0
  * * +   + 2
  *   *   + 1
  *     * + 1
    * * +   1
    *   *   0
      * * + 1
(2+1+1)+(2+1+1)+(1)+(1) = 10
(1+3)+(1+3)+(1)+(1) = 10
```

## Fast, abstract, mathematical
```C
int countTriplets(int* arr, int arrSize) {
    // calculate cummulative xor sum
    int *x;
    x = calloc(arrSize+1,sizeof(int));
    for(int i=0; i<arrSize; i++){
        x[i+1] = x[i]^arr[i];
    }

    int out;
    out=0;

    for(int i=0; i<arrSize; i++){
        for(int j=i+2; j<=arrSize; j++){
            if(x[i]==x[j]){
                out += j-i-1;
            }
        }
    }

    return out;
}
```
## Fast enough
```C
int countTriplets(int* arr, int arrSize) {
    // calculate cummulative xor sum
    int *x;
    x = calloc(arrSize+1,sizeof(int));
    for(int i=0; i<arrSize; i++){
        x[i+1] = x[i]^arr[i];
    }

    int out;
    out=0;

    for(int i=0; i<arrSize; i++){
        for(int j=i+1; j<=arrSize; j++){
            int a;
            a = x[j]^x[i];
            for(int k=j+1; k<=arrSize; k++){
                int b;
                b = x[k]^x[j];
                if(b==a){
                    /*
                    for(int l=i; l<j; l++){
                        printf("%d ",arr[l]);
                    }
                    printf("| ");
                    for(int l=j; l<k; l++){
                        printf("%d ",arr[l]);
                    }
                    printf("\n");
                    */
                    out++;
                }
            }
        }
    }
    return out;
}
```
# 2024-05-29
[1404. Number of Steps to Reduce a Number in Binary Representation to One](https://leetcode.com/problems/number-of-steps-to-reduce-a-number-in-binary-representation-to-one/)
Does tail call recursion even matter? Some compilers don't seem to support it...
Trying making a tail call recursion version doesn't seem to affect the memory usage at all...
(Or maybe it's already optimized by the compiler? Despite not being in the "tail call" form.)

As usual, Leetcode measurements seems inconclusive. Sometimes I can get 100% speed or 100% space.

## Tail-Call?
```C
int steps(char *s, int n, int c){
    if(n<=0){
        return c;
    }
    if(s[n]=='0'){
        while(s[n]=='0'){
            n--;
            c++;
        }
        return steps(s, n, c);
    }

    while(n>=0 && s[n]=='1'){
        n--;
        c++;
    }
    c++;
    if(n<0){
        return c;
    }
    s[n] = '1';
    return steps(s, n, c);
}


int numSteps(char* s) {
    return steps(s, strlen(s)-1, 0);
}
```
## Works, Straightforward
```C
int steps(char *s, int n){
    if(n<=0){
        return 0;
    }
    int c;
    c=0;
    if(s[n]=='0'){
        while(s[n]=='0'){
            n--;
            c++;
        }
        //s[n+1] = 0;
        //printf("%s\n",s);
        return c + steps(s, n);
    }

    while(n>=0 && s[n]=='1'){
        n--;
        c++;
    }
    c++;
    if(n<0){
        return c;
    }
    s[n] = '1';
    //s[n+1] = 0;
    //printf("%s\n",s);
    return c + steps(s, n);
}


int numSteps(char* s) {
    return steps(s, strlen(s)-1);
}
```
# 2024-05-28
[1208. Get Equal Substrings Within Budget](https://leetcode.com/problems/get-equal-substrings-within-budget/)

## Fast & Simple
```C
int equalSubstring(char* s, char* t, int maxCost) {
    int n;
    n = strlen(s)+1;

    int *cost;
    cost = calloc(n,sizeof(int));

    for(int i=0; i<n-1; i++){
        int a;
        a = s[i]-t[i];
        if(a<0) a = 0-a;
        cost[i+1] = a;
    }

    for(int i=1; i<n; i++){
        cost[i] = cost[i]+cost[i-1];
    }

    /*
    for(int i=0; i<n; i++){
        printf("%d ",cost[i]);
    }
    */

    int max;
    max = 0;
    for(int i=0; i<n; i++){
        for(int j=i+max; j<n; j++){
            if(cost[j]<=cost[i]+maxCost){
                if(max<j-i){
                    max = j-i;
                }
            } else {
                break;
            }
        }
    }

    return max;
}
```
# 2024-05-27
[1608. Special Array With X Elements Greater Than or Equal X](https://leetcode.com/problems/special-array-with-x-elements-greater-than-or-equal-x/)

## Simple
```C
int compare(int *a, int *b){
    return *b-*a;
}

int specialArray(int* nums, int numsSize) {
    qsort(nums,numsSize,sizeof(int),compare);
    int i;
    i=0;
    while(i<numsSize && nums[i]>=i+1){
        i++;
    }

    if(i==0) return -1;
    if(i<numsSize && nums[i]==i) return -1;
    return i;
}
```
# 2024-05-26
[552. Student Attendance Record II](https://leetcode.com/problems/student-attendance-record-ii/)

## Works, Math, Less Memory
```C
int checkRecord(int n) {
    if(n<=3){
        switch(n){
            case 1: return 3;
            case 2: return 8;
            case 3: return 19;
            default: return 1;
        }
    }

    uint c[] = {1,2,4,7};
    uint p[] = {0,1,4,12};
    uint q[] = {0,0,1,3};

    for(int i=4; i<=n; i++){
        c[i&3] = 2*c[(i-1)&3]+1000000007-c[i&3];
        c[i&3] = c[i&3]%1000000007;

        p[i&3] = c[(i-1)&3]+2*p[(i-1)&3]+1000000007-q[(i-2)&3];
        p[i&3] = p[i&3]%1000000007;

        q[i&3] = p[(i-1)&3]+1000000007-q[(i-1)&3]+1000000007-q[(i-2)&3];
        q[i&3] = q[i&3]%1000000007;
    }

    
    return (c[n&3]+p[n&3])%1000000007;
}
```
## Works, Math
```C
int checkRecord(int n) {
    if(n<=3){
        switch(n){
            case 1: return 3;
            case 2: return 8;
            case 3: return 19;
            default: return 1;
        }
    }
    
    uint *c, *p, *q;
    c = calloc(n+1,sizeof(uint));
    p = calloc(n+1,sizeof(uint));
    q = calloc(n+1,sizeof(uint));
    
    c[0]=1; c[1]=2; c[2]=4; c[3]=7;
    p[0]=0; p[1]=1; p[2]=4; p[3]=12;
    q[0]=0; q[1]=0; q[2]=1; q[3]=3;
    
    for(int i=4; i<=n; i++){
        c[i] = 2*c[i-1]+1000000007-c[i-4];
        c[i] = c[i]%1000000007;
        //while(c[i]>1000000007) c[i] -= 1000000007;
        
        p[i] = c[i-1]+2*p[i-1]+1000000007-q[i-2];
        p[i] = p[i]%1000000007;
        //while(p[i]>1000000007) p[i] -= 1000000007;
        
        q[i] = p[i-1]+1000000007-q[i-1]+1000000007-q[i-2];
        q[i] = q[i]%1000000007;
        //while(q[i]>1000000007) q[i] -= 1000000007;
    }
    
    return (c[n]+p[n])%1000000007;
}
```
## Draft
```C
/*int dfs(int n, int l, int x){
    if(x>=n){
        return 0;
    }
    int sum;
    sum = 0;
    
    // 0L
    sum += n-l; // PPP... or A+PP...
    sum = sum%1000000007;

    // 1L
    sum += dfs(n,l+1,x+2); // LP...
    sum = sum%1000000007;

    // 2L
    sum += dfs(n,l+2,x+3); // LLP...
    sum = sum%1000000007;

    // 3L
    sum += dfs(n,l+3,x+4); // LLLP...
    sum = sum%1000000007;

    return sum;
}
*/

int dfs(int n, bool *b, int *a){
    return 0;
}

int checkRecord(int n) {
    bool *b;
    b = calloc(n+1,sizeof(bool));
    int *a;
    a = calloc(n+1,sizeof(int));
    //b[0] = b[1] = b[2] = b[3] = true;
    a[0] = 0;
    //return dfs(n,0,0);
    //return dfs(n,b,a);
    return 0;
}
```
# 2024-05-25
[140. Word Break II](https://leetcode.com/problems/word-break-ii/)

## Slow but works.
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

struct list{
    char *s;
    struct list *next;
};

struct list* dfs(char* s, int len, char** dict, int n, struct list *l, char *t, int p, int x){
    if(x>=len){
        l->s = malloc((p+1)*sizeof(char));
        strcpy(l->s,t+1);
        l->next = calloc(1,sizeof(struct list));
        return l->next;
    }

    for(int i=0; i<n; i++){
        int j;
        j=0;
        while(s[x+j] && dict[i][j] && s[x+j]==dict[i][j]){
            j++;
        }
        if(dict[i][j]==0){
            t[p] = ' ';
            strcpy(&t[p+1],dict[i]);
            l = dfs(s,len,dict,n,l,t,p+1+j,x+j);
        }
    }

    return l;
}

char** wordBreak(char* s, char** wordDict, int wordDictSize, int* returnSize) {
    struct list *l;
    char *t;
    int size;

    l = calloc(1,sizeof(struct list));
    t = calloc(strlen(s)*2+2,sizeof(char));
    dfs(s, strlen(s), wordDict, wordDictSize, l, t, 0, 0);

    struct list *m;
    m = l;
    int count;
    count = 0;
    while(m){
        //if(m->s) printf("%s\n",m->s);
        count++;
        m = m->next;
    }
    count--;

    char **out;
    out = malloc(count*sizeof(char*));
    for(int i=0; i<count; i++){
        out[i] = l->s;
        l = l->next;
    }

    *returnSize = count;
    return out;
}
```
# 2024-05-24
[1255. Maximum Score Words Formed by Letters](https://leetcode.com/problems/maximum-score-words-formed-by-letters/)

Similar structure to yesterday's problem. Solved with DFS approach. Incorporates the technique of reversing things... (Backtracking.)
## DFS
```C
int dfs(int **wlc, int n, int *lc, int *s, int x){
    if(x>=n){
        return 0;
    }

    int max;
    max = 0;

    int a;
    // skip word
    a = dfs(wlc,n,lc,s,x+1);
    if(max<a) max=a;

    // test word
    for(int i=0; i<26; i++){
        if(lc[i]<wlc[x][i]){
            return max;
        }
    }
    int score;
    score = 0;
    for(int i=0; i<26; i++){
        lc[i] -= wlc[x][i];
        score += wlc[x][i]*s[i];
    }
    a = score+dfs(wlc,n,lc,s,x+1);
    if(max<a) max=a;

    // reverse
    for(int i=0; i<26; i++){
        lc[i] += wlc[x][i];
    }

    return max;
}

int maxScoreWords(char** words, int wordsSize, char* letters, int lettersSize, int* score, int scoreSize) {
    
    // count letters
    int *lcount;
    lcount = calloc(26,sizeof(int));

    for(int i=0; i<lettersSize; i++){
        lcount[letters[i]-'a']++;
    }
    /*
    for(int i=0; i<26; i++){
        if(lcount[i]){
            printf("%c:%d ",'a'+i,lcount[i]);
        }
    }
    */

    int **wlcount;
    wlcount = malloc(wordsSize*sizeof(int*));
    for(int i=0; i<wordsSize; i++){
        wlcount[i] = calloc(26,sizeof(int));
        char *a;
        a = words[i];
        while(a[0]){
            wlcount[i][a[0]-'a']++;
            a++;
        }
    }

    /*
    printf("\n");
    for(int i=0; i<wordsSize; i++){
        printf("%s; ",words[i]);
        for(int j=0; j<26; j++){
            if(wlcount[i][j]){
                printf("%c:%d ",'a'+j,wlcount[i][j]);
            }
        }
        printf("\n");
    }
    */

    return dfs(wlcount,wordsSize,lcount,score,0);
}
```
# 2024-05-23
[2597. The Number of Beautiful Subsets](https://leetcode.com/problems/the-number-of-beautiful-subsets/)

## DFS
```C
int compare(int *a, int *b){
    return *a - *b;
}

int dfs(int *nums, int n, int k, bool *b, int x){
    if(x>=n){
        return 1;
    }
    if(b[x]){
        return dfs(nums,n,k,b,x+1);
    }
    
    int sum = 0;

    // not choose x
    //printf("~%d ",nums[x]);
    sum += dfs(nums,n,k,b,x+1);

    // choose x
    int a;
    a = nums[x]+k;
    int i;
    i = x+1;
    while(i<n && nums[i]<=a){
        if(nums[i]==a){
            b[i] = true;
        }
        i++;
    }
    //printf("%d ",nums[x]);
    sum += dfs(nums,n,k,b,x+1);

    // reverse
    i = x+1;
    while(i<n && nums[i]<=a){
        if(nums[i]==a){
            b[i] = false;
        }
        i++;
    }

    return sum;
}

int beautifulSubsets(int* nums, int numsSize, int k) {
    qsort(nums,numsSize,sizeof(int),compare);

    /*
    for(int i=0; i<numsSize; i++){
        printf("%d ",nums[i]);
    }
    */

    bool *b;
    b = calloc(numsSize,sizeof(bool));
    return dfs(nums,numsSize,k,b,0)-1;
}
```
# 2024-05-22
[131. Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)
"Medium" but really tedious implementation.

## Tedious
```C
/**
 * Return an array of arrays of size *returnSize.
 * The sizes of the arrays are returned as *returnColumnSizes array.
 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
 */

struct sub{
    char *c;
    int s;
    int e;
    struct sub *next;
};

struct part{
    char **str;
    int len;
    struct part *next;
};

struct part * dfs(struct sub **head, int n, struct sub **list, struct part *p, int x, int c){
    if(x>=n){
        char **str;
        str = malloc(c*sizeof(char*));
        for(int i=0; i<c; i++){
            str[i] = list[i]->c;
        }
        struct part *p2;
        p2 = malloc(sizeof(struct part));
        p2->str = str;
        p2->len = c;
        p2->next = NULL;
        p->next = p2;
        return p2;
    }

    struct sub *curr;
    curr = head[x];
    while(curr){
        list[c] = curr;
        p = dfs(head,n,list,p,(curr->e)+1,c+1);
        curr = curr->next;
    }

    return p;
}

char*** partition(char* s, int* returnSize, int** returnColumnSizes) {
    int n;
    n = strlen(s);

    // find substrings which are palindromes
    // head[i] are substrings starting at index i
    struct sub **head;
    head = calloc(n,sizeof(struct sub *));

    char *a;
    a = calloc(n,sizeof(char));
    
    for(int i=0; i<n; i++){
        struct sub *curr;
        curr = malloc(sizeof(struct sub));
        head[i] = curr;

        for(int j=i; j<n; j++){
            if(j==i){
                char *b;
                b = calloc(2,sizeof(char));
                strncpy(b,&s[i],1);
                curr->c = b;
                curr->s = i;
                curr->e = j;
                curr->next = NULL;
            }else{
                int k, l;
                k=i;
                l=j;
                bool p;
                p = true;
                while(k<l){
                    if(s[k]!=s[l]){
                        p = false;
                        break;
                    }
                    k++;
                    l--;
                }
                if(p){
                    struct sub *next;
                    next = malloc(sizeof(struct sub));
                    
                    char *b;
                    b = calloc(j-i+2,sizeof(char));
                    strncpy(b,&s[i],j-i+1);
                    next->c = b;
                    next->s = i;
                    next->e = j;
                    next->next = NULL;
                    curr->next = next;
                    curr = next;
                }
            }
        }
    }

    for(int i=0; i<n; i++){
        struct sub *curr;
        curr = head[i];
        while(curr){
            printf("%s ",curr->c);
            curr = curr->next;
        }
        printf("\n");
    }

    // create partitions
    struct part *p;
    p = malloc(sizeof(struct part));
    p->str = NULL;
    p->len = 0;
    p->next = NULL;
    struct sub **list;
    list = calloc(n,sizeof(struct sub *));
    dfs(head,n,list,p,0,0);

    /*
    while(p){
        printf(": ");
        for(int i=0; i<p->len; i++){
            printf("%s ",(p->str)[i]);
        }
        printf("\n");
        p = p->next;
    }
    */

    // convert to output
    struct part *pp;
    pp = p;
    int len;
    len = 0;
    while(pp){
        len++;
        pp = pp->next;
    }
    len--;

    char ***out;
    out = malloc(len*sizeof(char **));
    int *colSizes;
    colSizes = malloc(len*sizeof(int));
    p = p->next;
    for(int i=0; i<len; i++){
        //printf(".");
        out[i] = p->str;
        colSizes[i] = p->len;
        p = p->next;
    }

    *returnSize = len;
    *returnColumnSizes = colSizes;
    return out;
}
```

# 2024-05-21
[78. Subsets](https://leetcode.com/problems/subsets/)
Two pass approach. First to get sizes and allocate arrays. Second to fill them in.
Leetcode timing and memory measurement can be a little inaccurate... There can be a lot of variance...
With 3 runs:
- 6ms, 7ms, 2ms
- 6MB, 6.3MB, 6.4MB
It seems difficult to get it any faster...
Though a way is to reduce the amount of calloc and malloc calls...
## Straight Forward
```C
/**
 * Return an array of arrays of size *returnSize.
 * The sizes of the arrays are returned as *returnColumnSizes array.
 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
 */
int** subsets(int* nums, int numsSize, int* returnSize, int** returnColumnSizes) {
    int n;
    n = 1<<numsSize;
    *returnSize = n;
    *returnColumnSizes = calloc(n,sizeof(int));

    int **out;
    out = calloc(n,sizeof(int*));

    for(int i=1; i<n; i++){
        int j, c;
        j = i;
        c = 0;
        while(j>0){
            switch(j&7){
                case 1:
                case 2:
                case 4:
                    c+=1;
                    break;
                case 3:
                case 5:
                case 6:
                    c+=2;
                    break;
                case 7:
                    c+=3;
                    break;
                case 0:
                default:
                    break;
            }
            j = j>>3;
        }
        (*returnColumnSizes)[i] = c;
        out[i] = calloc(c,sizeof(int));
    }

    for(int i=1; i<n; i++){
        int j, c, s;
        j = i;
        c = 0;
        s = 0;
        while(j>0){
            switch(j&7){
                case 1:
                    out[i][c] = nums[s]; c++;
                    break;
                case 3:
                    out[i][c] = nums[s]; c++;
                case 2:
                    out[i][c] = nums[s+1]; c++;
                    break;
                case 5:
                    out[i][c] = nums[s]; c++;
                case 4:
                    out[i][c] = nums[s+2]; c++;
                    break;
                case 7:
                    out[i][c] = nums[s]; c++;
                case 6:
                    out[i][c] = nums[s+1]; c++;
                    out[i][c] = nums[s+2]; c++;
                    break;
                default:
                    break;
            }
            j = j>>3;
            s += 3;
        }
    }

    return out;
}
```
# 2024-05-20
[1863. Sum of All Subset XOR Totals](https://leetcode.com/problems/sum-of-all-subset-xor-totals/)

## Simple
```C
int subsetXORSum(int* nums, int numsSize) {
    int n;
    n = 1<<numsSize;

    int sum;
    sum = 0;

    for(int i=1; i<n; i++){
        int j, c, xor;
        j = i;
        c = 0;
        xor = 0;
        while(j){
            if(j&1){
                xor = xor ^ nums[c];
            }
            c++;
            j = j>>1;
        }
        sum += xor;
    }
    return sum;
}
```
# 2024-05-19
[3068. Find the Maximum Sum of Node Values](https://leetcode.com/problems/find-the-maximum-sum-of-node-values/)
Solution: Each node could either be "odd" or "even". Odd nodes are nodes that when flipped the sum will increase. We keep a count of odd nodes. Via induction, we can flip any pair of nodes (in a tree) such that we will end up with a single odd node, or zero odd node. Thus, we keep the max sum and keep a minimum difference. If we have an odd number of odd nodes, we return max-min. Else if even, we return max.
## Elegant, Mathematical, O(1)
```C
long long maximumValueSum(int* nums, int numsSize, int k, int** edges, int edgesSize, int* edgesColSize) {
    long long max;
    int min;
    int oddc;
    oddc = 0;
    max = 0;
    min = nums[0] - (k^nums[0]);
    if(min<0){
        min = 0-min;
    }
    for(int i=0; i<numsSize; i++){
        int xor, dif;
        xor = k^nums[i];
        if(xor>nums[i]){
            dif = xor-nums[i];
            if(dif<min) min = dif;
            max += xor;
            oddc++;
        }else{
            dif = nums[i]-xor;
            if(dif<min) min = dif;
            max += nums[i];
        }
    }
    if((oddc&1)==0){
        return max;
    }else{
        return max-min;
    }
}
```
## Initial Draft
```C
long long maximumValueSum(int* nums, int numsSize, int k, int** edges, int edgesSize, int* edgesColSize) {
    long long sum;

    int *xor, *dif;
    bool *odd;
    int oddc;
    oddc = 0;
    xor = calloc(numsSize,sizeof(int));
    dif = calloc(numsSize,sizeof(int));
    odd = calloc(numsSize,sizeof(bool));
    sum = 0;
    for(int i=0; i<numsSize; i++){
        xor[i] = k^nums[i];
        if(xor[i]>nums[i]){
            dif[i] = xor[i]-nums[i];
            odd[i] = true;
            oddc++;
        }else{
            dif[i] = nums[i]-xor[i];
        }
        sum += nums[i];
    }
    if(oddc==0){
        return sum;
    }

    long long max;
    max = sum;

    /*
    int *count;
    count = calloc(numsSize,sizeof(int));
    for(int i=0; i<edgesSize; i++){
        count[edges[i][0]]++;
        count[edges[i][1]]++;
    }
    */

    

    bool *flip;
    flip = calloc(edgesSize,sizeof(bool));


    return 0;

}
```
# 2024-05-18
[979. Distribute Coins in Binary Tree](https://leetcode.com/problems/distribute-coins-in-binary-tree/)

## Works Decent
```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
int dfs(struct TreeNode *n, int *m){
    int a;
    a = n->val - 1;

    if(n->left){
        a += dfs(n->left,m);
    }
    if(n->right){
        a += dfs(n->right,m);
    }
    
    if(a<0){
        *m -= a;
    }else{
        *m += a;
    }

    return a;
}

int distributeCoins(struct TreeNode* root) {
    int a, b;
    b = 0;

    a = dfs(root,&b);

    printf("%d %d\n",a,b);
    return b;
}
```

# 2024-05-17
[1325. Delete Leaves With a Given Value](https://leetcode.com/problems/delete-leaves-with-a-given-value/)

## Works
```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */

bool dfs(struct TreeNode* n, int target){
    if(n->left != NULL && dfs(n->left, target)){
        n->left = NULL;
    }

    if(n->right != NULL && dfs(n->right, target)){
        n->right = NULL;
    }

    if(n->left == NULL && n->right == NULL && n->val == target){
        return true;
    }

    return false;
}

struct TreeNode* removeLeafNodes(struct TreeNode* root, int target) {
    bool b;
    b = dfs(root,target);

    if(b){
        return NULL;
    } else {
        return root;
    }
}
```

# 2024-05-16
[2331. Evaluate Boolean Binary Tree](https://leetcode.com/problems/evaluate-boolean-binary-tree/)

```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
bool evaluateTree(struct TreeNode* root) {
    if(root->val == 0) return false;
    if(root->val == 1) return true;
    if(root->val == 2) {
        return evaluateTree(root->left) || evaluateTree(root->right);
    }

    return evaluateTree(root->left) && evaluateTree(root->right);
}
```
# 2024-05-15
[2812. Find the Safest Path in a Grid](https://leetcode.com/problems/find-the-safest-path-in-a-grid/)
Finally implemented Union-Find. Looked at Wikipedia article on it... Really "simple and easy"... But it's really elegant. It's actually dumb how schools don't teach it. It's a must have... Very fundamental stuff.
But yeah, Union-Find can be implemented even without ranks and sizes.

![[Pasted image 20240516030143.png]]

![[Pasted image 20240516030106.png]]
## Union-Find
```C
int find(int* p, int g){
    while(g != p[g]){
        g = p[g];
    }
    return g;
}

void make(int* p, int g){
    p[g] = g;
}

void merge(int *p, int a, int b){
    a = find(p,a);
    b = find(p,b);

    if(a==b) return;

    if(a<b){
        p[b] = a;
    } else {
        p[a] = b;
    }
}

int maximumSafenessFactor(int** grid, int gridSize, int* gridColSize){
    // easy edge cases
    if(grid[0][0]) return 0;
    if(grid[gridSize-1][gridSize-1]) return 0;

    int** sf;
    sf = malloc(sizeof(int*)*gridSize);
    for(int i=0; i<gridSize; i++){
        sf[i] = calloc(gridSize,sizeof(int));
    }

    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize; j++){
            sf[i][j] = 400;
        }
    }

    // calculate safety
    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize; j++){
            if(grid[i][j]){
                sf[i][j] = 0;
                for(int k=i-1; k>=0; k--){
                    int a = sf[k+1][j]+1;
                    if(a<sf[k][j]){ sf[k][j] = a; } else { break; }
                }
                for(int k=i+1; k<gridSize; k++){
                    int a = sf[k-1][j]+1;
                    if(a<sf[k][j]){ sf[k][j] = a; } else { break; }
                }
                for(int l=j+1; l<gridSize; l++){
                    int b;
                    b = sf[i][l-1]+1;
                    if(b<sf[i][l]){
                        sf[i][l] = b;
                        for(int k=i-1; k>=0; k--){
                            int a = sf[k+1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                        for(int k=i+1; k<gridSize; k++){
                            int a = sf[k-1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                    } else {
                        break;
                    }
                }
                for(int l=j-1; l>=0; l--){
                    int b;
                    b = sf[i][l+1]+1;
                    if(b<sf[i][l]){
                        sf[i][l] = b;
                        for(int k=i-1; k>=0; k--){
                            int a = sf[k+1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                        for(int k=i+1; k<gridSize; k++){
                            int a = sf[k-1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                    } else {
                        break;
                    }
                }
            }
        }
    }

    /*
    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize; j++){
            if(sf[i][j]>2)
            printf("%c ",'0'+sf[i][j]);
            else
            printf("  ");
        }
        printf("\n");
    }
    */

    int out;
    int max, max2;
    max = sf[0][0];
    for(int i=0; i<gridSize; i++){
        max2 = sf[i][0];
        for(int j=0; j<=i; j++){
            int b = sf[i-j][j];
            if(b>max2){
                max2 = b;
            }
        }
        if(max2 < max){
            max = max2;
        }
    }
    for(int i=1; i<gridSize; i++){
        max2 = sf[i][gridSize-1];
        for(int j=0; j<gridSize-i; j++){
            int b = sf[i+j][gridSize-1-j];
            if(b>max2){
                max2 = b;
            }
        }
        if(max2 < max){
            max = max2;
        }
    }
    out = max;

    int **g;
    g = malloc(sizeof(int*)*gridSize);
    for(int i=0; i<gridSize; i++){
        g[i] = calloc(gridSize,sizeof(int));
    }
    int *p;
    p = calloc(gridSize*gridSize,sizeof(int));
    for(int i=0; i<gridSize*gridSize; i++){
        p[i] = i;
    }

    int cg, v;
    cg = 1;
    v = out;
    while(v>0){
        for(int i=0; i<gridSize; i++){
            for(int j=0; j<gridSize; j++){
                if(sf[i][j]>=v){
                    if(i==0){
                        if(j>0 && g[i][j-1]){
                            g[i][j] = g[i][j-1];
                        } else {
                            g[i][j] = cg;
                            make(p,cg);
                            cg++;
                        }
                    } else {
                        if(j>0 && g[i][j-1]){
                            if(g[i-1][j]){
                                g[i][j] = g[i-1][j];
                                merge(p,g[i][j-1],g[i-1][j]);
                            } else {
                                g[i][j] = g[i][j-1];
                            }
                        } else if(g[i-1][j]){
                            g[i][j] = g[i-1][j];
                        } else {
                            g[i][j] = cg;
                            make(p,cg);
                            cg++;
                        }
                    }
                    
                }
            }
        }
        if(find(p,g[0][0]) == find(p,g[gridSize-1][gridSize-1])){
            /*
            for(int i=0; i<gridSize; i++){
                for(int j=0; j<gridSize; j++){
                    if(g[i][j]){
                    printf("%3d ",find(p,g[i][j]));
                    } else {
                        printf("    ");
                    }
                }
                printf("\n");
            }
            */
            return v;
        }
        v--;
    }
    return 0;
}
```

## Fail 2, Too Long
```C

bool reach(bool **up, bool **left, int y, int x){
    if(x==0 && y==0){
        return true;
    }

    bool out;
    out = false;

    if(y>0 && up[y][x]){
        out = out || reach(up, left, y-1, x);
    }

    if(x>0 && left[y][x]){
        out = out || reach(up, left, y, x-1);
    }

    return out;
}

int maximumSafenessFactor(int** grid, int gridSize, int* gridColSize){
    // easy edge cases
    if(grid[0][0]) return 0;
    if(grid[gridSize-1][gridSize-1]) return 0;

    int** sf;
    sf = malloc(sizeof(int*)*gridSize);
    for(int i=0; i<gridSize; i++){
        sf[i] = calloc(gridSize,sizeof(int));
    }

    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize; j++){
            sf[i][j] = 400;
        }
    }

    //printf("reach");

    // calculate safety
    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize; j++){
            if(grid[i][j]){
                sf[i][j] = 0;
                for(int k=i-1; k>=0; k--){
                    int a = sf[k+1][j]+1;
                    if(a<sf[k][j]){ sf[k][j] = a; } else { break; }
                }
                for(int k=i+1; k<gridSize; k++){
                    int a = sf[k-1][j]+1;
                    if(a<sf[k][j]){ sf[k][j] = a; } else { break; }
                }
                for(int l=j+1; l<gridSize; l++){
                    int b;
                    b = sf[i][l-1]+1;
                    if(b<sf[i][l]){
                        sf[i][l] = b;
                        for(int k=i-1; k>=0; k--){
                            int a = sf[k+1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                        for(int k=i+1; k<gridSize; k++){
                            int a = sf[k-1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                    } else {
                        break;
                    }
                }
                for(int l=j-1; l>=0; l--){
                    int b;
                    b = sf[i][l+1]+1;
                    if(b<sf[i][l]){
                        sf[i][l] = b;
                        for(int k=i-1; k>=0; k--){
                            int a = sf[k+1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                        for(int k=i+1; k<gridSize; k++){
                            int a = sf[k-1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                    } else {
                        break;
                    }
                }
            }
        }
    }

    /*
    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize; j++){
            printf("%d ",sf[i][j]);
        }
        printf("\n");
    }
    */

    bool **up, **left;
    up = malloc(sizeof(bool*)*gridSize);
    for(int i=0; i<gridSize; i++){
        up[i] = calloc(gridSize,sizeof(bool));
    }
    left = malloc(sizeof(bool*)*gridSize);
    for(int i=0; i<gridSize; i++){
        left[i] = calloc(gridSize,sizeof(bool));
    }

    int out;
    out = sf[0][0];
    if(sf[gridSize-1][gridSize-1] < out){
        out = sf[gridSize-1][gridSize-1];
    }
    while(out>0){
        for(int j=0; j<gridSize; j++){
            for(int k=0; k<gridSize; k++){
                if(k>0 && sf[j][k-1] >= out){
                    left[j][k] = true;
                }
                if(j>0 && sf[j-1][k] >= out){
                    up[j][k] = true;
                }
            }
        }
        if(reach(up,left,gridSize-1,gridSize-1)){
            return out;
        }
        out--;
    }

    return 0;
}
```
## Fail 1, Wrong Logic... (Line 112)
```C
bool reach(bool **up, bool **left, int y, int x){
    if(x==0 && y==0){
        return true;
    }

    bool out;
    out = false;

    if(y>0 && up[y][x]){
        out = out || reach(up, left, y-1, x);
    }

    if(x>0 && left[y][x]){
        out = out || reach(up, left, y, x-1);
    }

    return out;
}

int maximumSafenessFactor(int** grid, int gridSize, int* gridColSize){
    // easy edge cases
    if(grid[0][0]) return 0;
    if(grid[gridSize-1][gridSize-1]) return 0;

    int** sf;
    sf = malloc(sizeof(int*)*gridSize);
    for(int i=0; i<gridSize; i++){
        sf[i] = calloc(gridSize,sizeof(int));
    }

    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize; j++){
            sf[i][j] = 400;
        }
    }

    //printf("reach");

    // calculate safety
    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize; j++){
            if(grid[i][j]){
                sf[i][j] = 0;
                for(int k=i-1; k>=0; k--){
                    int a = sf[k+1][j]+1;
                    if(a<sf[k][j]){ sf[k][j] = a; } else { break; }
                }
                for(int k=i+1; k<gridSize; k++){
                    int a = sf[k-1][j]+1;
                    if(a<sf[k][j]){ sf[k][j] = a; } else { break; }
                }
                for(int l=j+1; l<gridSize; l++){
                    int b;
                    b = sf[i][l-1]+1;
                    if(b<sf[i][l]){
                        sf[i][l] = b;
                        for(int k=i-1; k>=0; k--){
                            int a = sf[k+1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                        for(int k=i+1; k<gridSize; k++){
                            int a = sf[k-1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                    } else {
                        break;
                    }
                }
                for(int l=j-1; l>=0; l--){
                    int b;
                    b = sf[i][l+1]+1;
                    if(b<sf[i][l]){
                        sf[i][l] = b;
                        for(int k=i-1; k>=0; k--){
                            int a = sf[k+1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                        for(int k=i+1; k<gridSize; k++){
                            int a = sf[k-1][l]+1;
                            if(a<sf[k][l]){ sf[k][l] = a; } else { break; }
                        }
                    } else {
                        break;
                    }
                }
            }
        }
    }

    /*
    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize; j++){
            printf("%d ",sf[i][j]);
        }
        printf("\n");
    }
    */

    bool **up, **left;
    up = malloc(sizeof(bool*)*gridSize);
    for(int i=0; i<gridSize; i++){
        up[i] = calloc(gridSize,sizeof(bool));
    }
    left = malloc(sizeof(bool*)*gridSize);
    for(int i=0; i<gridSize; i++){
        left[i] = calloc(gridSize,sizeof(bool));
    }

    int out;
    out = sf[0][0];
    if(sf[gridSize-1][gridSize-1] > out){
        out = sf[gridSize-1][gridSize-1];
    }
    while(out>0){
        for(int j=0; j<gridSize; j++){
            for(int k=0; k<gridSize; k++){
                if(k>0 && sf[j][k-1] >= out){
                    left[j][k] = true;
                }
                if(j>0 && sf[j-1][k] >= out){
                    up[j][k] = true;
                }
            }
        }
        if(reach(up,left,gridSize-1,gridSize-1)){
            return out;
        }
        out--;
    }

    return 0;
}
```
# 2024-05-14
[1219. Path with Maximum Gold](https://leetcode.com/problems/path-with-maximum-gold/)
Rather than creating a new "reach" array for each step. Each step is backtracked instead. Works since it's DFS.
## Not Cheating (DFS)
```C
int gold(int **grid, int h, int w, int y, int x, int g, int *r) {
    if(y<0 || y>=h || x<0 || x>=w){
        return g;
    }

    if(r[y]&(1<<x)){
        return g;
    }
    
    int max, gold2, next;
    max = 0;
    gold2 = grid[y][x];
    r[y] |= (1<<x);

    next = gold(grid,h,w,y-1,x,g+gold2,r);
    if(max < next) max = next;

    next = gold(grid,h,w,y+1,x,g+gold2,r);
    if(max < next) max = next;

    next = gold(grid,h,w,y,x-1,g+gold2,r);
    if(max < next) max = next;

    next = gold(grid,h,w,y,x+1,g+gold2,r);
    if(max < next) max = next;

    r[y] &= ~(1<<x);

    return max;
}

int getMaximumGold(int** grid, int gridSize, int* gridColSize) {
    int max, h, w, g;
    max = 0;
    h = gridSize;
    w = gridColSize[0];

    bool hasZero;
    hasZero = false;

    int* r;
    r = calloc(h,sizeof(int));
    for(int i=0; i<h; i++){
        for(int j=0; j<w; j++){
            if(grid[i][j]==0){
                r[i] |= (1<<j);
                hasZero = true;
            }
        }
    }

    if(!hasZero){
        int sum;
        sum = 0;
        for(int i=0; i<h; i++){
            for(int j=0; j<w; j++){
                sum += grid[i][j];
            }
        }
        return sum;
    }

    for(int i=0; i<h; i++){
        for(int j=0; j<w; j++){
            g = gold(grid,h,w,i,j,0,r);
            if(g>max){
                max = g;
            }
        }
    }
    return max;
}
```
## Cheated to Pass
```C
int gold(int **grid, int h, int w, int y, int x, int g, int *r) {
    if(y<0 || y>=h || x<0 || x>=w){
        return g;
    }

    if(r[y]&(1<<x)){
        return g;
    }

    //printf("%d ",grid[y][x]);

    int max, gold2, next;
    max = 0;
    gold2 = grid[y][x];
    //grid[y][x] = 0;
    int *r2;
    r2 = calloc(h,sizeof(int));

    for(int i=0; i<h; i++) r2[i] = r[i];
    r2[y] |= (1<<x);

    next = gold(grid,h,w,y-1,x,g+gold2,r2);
    if(max < next) {
        //printf("* ");
        max = next;
    }
    for(int i=0; i<h; i++) r2[i] = r[i];
    r2[y] |= (1<<x);

    next = gold(grid,h,w,y+1,x,g+gold2,r2);
    if(max < next) {
        //printf("* ");
        max = next;
    }
    for(int i=0; i<h; i++) r2[i] = r[i];
    r2[y] |= (1<<x);

    next = gold(grid,h,w,y,x-1,g+gold2,r2);
    if(max < next) {
        //printf("* ");
        max = next;
    }
    for(int i=0; i<h; i++) r2[i] = r[i];
    r2[y] |= (1<<x);

    next = gold(grid,h,w,y,x+1,g+gold2,r2);
    if(max < next) {
        //printf("* ");
        max = next;
    }

    free(r2);

    return max;
}

int getMaximumGold(int** grid, int gridSize, int* gridColSize) {
    int max, h, w, g;
    max = 0;
    h = gridSize;
    w = gridColSize[0];

    bool hasZero;
    hasZero = false;

    int* r;
    r = calloc(h,sizeof(int));
    for(int i=0; i<h; i++){
        for(int j=0; j<w; j++){
            if(grid[i][j]==0){
                r[i] |= (1<<j);
                hasZero = true;
            }
        }
    }

    if(!hasZero){
        int sum;
        sum = 0;
        for(int i=0; i<h; i++){
            for(int j=0; j<w; j++){
                sum += grid[i][j];
            }
        }
        return sum;
    }

    for(int i=0; i<h; i++){
        for(int j=0; j<w; j++){
            g = gold(grid,h,w,i,j,0,r);
            if(g>max){
                max = g;
            }
        }
    }
    return max;
}
```
## Fail
```C
int gold(int **grid, int h, int w, int y, int x, int g) {
    if(y<0 || y>=h || x<0 || x>=w){
        return g;
    }

    if(grid[y][x] == 0){
        return g;
    }

    int max, gold2, next;
    max = 0;
    gold2 = grid[y][x];
    grid[y][x] = 0;
    
    next = gold(grid,h,w,y-1,x,g+gold2);
    if(max < next) max = next;
    next = gold(grid,h,w,y+1,x,g+gold2);
    if(max < next) max = next;
    next = gold(grid,h,w,y,x-1,g+gold2);
    if(max < next) max = next;
    next = gold(grid,h,w,y,x+1,g+gold2);
    if(max < next) max = next;

    return max;
}

int getMaximumGold(int** grid, int gridSize, int* gridColSize) {
    int max, h, w, g;
    max = 0;
    h = gridSize;
    w = gridColSize[0];

    int** grid2;
    grid2 = malloc(sizeof(int*)*gridSize);
    for(int i=0; i<gridSize; i++){
        grid2[i] = calloc(gridColSize[0],sizeof(int));
    }

    for(int i=0; i<h; i++){
        for(int j=0; j<w; j++){
            for(int k=0; k<h; k++){
                for(int l=0; l<w; l++){
                    grid2[k][l] = grid[k][l];
                }
            }
            
            g = gold(grid2,h,w,i,j,0);
            if(g>max){
                max = g;
            }
        }
    }
    return max;
}
```

# 2024-05-13
[861. Score After Flipping Matrix](https://leetcode.com/problems/score-after-flipping-matrix/)

## Basic Greedy
```C
int matrixScore(int** grid, int gridSize, int* gridColSize) {
    // greedy
    for(int i=0; i<gridSize; i++){
        if(grid[i][0]==0){
            for(int j=0; j<gridColSize[i]; j++){
                grid[i][j] ^= 1;
            }
        }
    }

    int score, sh;
    sh = gridColSize[0]-1;
    score = (1<<sh)*gridSize;
    // count
    for(int i=1; i<gridColSize[0]; i++){
        int c;
        c = 0;
        for(int j=0; j<gridSize; j++){
            c += grid[j][i];
        }
        if(c < gridSize-c){
            c = gridSize-c;
        }
        sh--;
        score += (1<<sh)*c;
    }

    return score;
}
```
# 2024-05-12
[2373. Largest Local Values in a Matrix](https://leetcode.com/problems/largest-local-values-in-a-matrix/)
There is some variance in the timing and memory usage... Not really much to optimize unfortunately. (Best possible solution. Unless someone can figure out how to compare values without reading a particular value...)
## Straight forward
```C
/**
 * Return an array of arrays of size *returnSize.
 * The sizes of the arrays are returned as *returnColumnSizes array.
 * Note: Both returned array and *columnSizes array must be malloced, assume caller calls free().
 */

int largest3(int a, int b, int c){
    if(a>b){
        if(a>c){
            return a;
        } else {
            return c;
        }
    } else if(b>c){
        return b;
    } else {
        return c;
    }
}

int** largestLocal(int** grid, int gridSize, int* gridColSize, int* returnSize, int** returnColumnSizes) {
    /*
    for(int i=0; i<gridSize; i++){
        for(int j=0; j<gridSize-2; j++){
            grid[i][j] = largest3(grid[i][j],grid[i][j+1],grid[i][j+2]);
        }
    }
    */

    for(int i=0; i<gridSize-2; i++){
        for(int j=0; j<gridSize-2; j++){
            grid[i][j] = largest3(
                largest3(grid[i][j],grid[i+1][j],grid[i+2][j]),
                largest3(grid[i][j+1],grid[i+1][j+1],grid[i+2][j+1]),
                largest3(grid[i][j+2],grid[i+1][j+2],grid[i+2][j+2])
                );
        }
    }

    *returnSize = gridSize-2;
    *returnColumnSizes = calloc(gridSize-2,sizeof(int));
    for(int i=0; i<gridSize-2; i++){
        returnColumnSizes[0][i] = gridSize-2;
    }
    return grid;
}
```
# 2024-05-11
[857. Minimum Cost to Hire K Workers](https://leetcode.com/problems/minimum-cost-to-hire-k-workers/)
Couldn't get the medal in time... Though got it solved.

## Greedy Search
```C
struct worker {
    int q;
    int w;
    double wpq;
    int ind;
};

struct quality {
    int q;
    bool b;
};

int compareQuality(struct worker *a, struct worker *b){
    return a->q - b->q;
}

int compareWPQ(struct worker *a, struct worker *b){
    if(a->wpq > b->wpq){
        return 1;
    } else if(a->wpq == b->wpq){
        return 0;
    } else {
        return -1;
    }
}

int compareWage(struct worker *a, struct worker *b){
    return a->w - b->w;
}

double mincostToHireWorkers(int* quality, int qualitySize, int* wage, int wageSize, int k) {
    struct worker *w;
    w = malloc(sizeof(struct worker)*wageSize);
    
    for(int i=0; i<wageSize; i++){
        w[i].q = quality[i];
        w[i].w = wage[i];
        w[i].wpq = (double)wage[i]/quality[i];
    }

    if(k==1){
        qsort(w,wageSize,sizeof(struct worker),compareWage);
        double o;
        o = w[0].w;
        return o;
    }

    // sort by quality
    struct quality *quals;
    quals = calloc(wageSize, sizeof(struct quality));
    
    qsort(w,wageSize,sizeof(struct worker),compareQuality);
    for(int i=0; i<wageSize; i++){
        w[i].ind = i;
        quals[i].q = w[i].q;
        
    }

    // sort by WPQ
    double out, wpq, cost;
    int qsum;
    //out = 1E+37;
    int qmax;
    qmax = 0;
    qsort(w,wageSize,sizeof(struct worker),compareWPQ);
    qsum = 0;
    for(int i=0; i<k; i++){
        qsum += w[i].q;
        quals[w[i].ind].b = true;
        if(qmax < w[i].ind){
            qmax = w[i].ind;
        }
    }
    wpq = w[k-1].wpq;
    cost = wpq*qsum;
    out = cost;

    // search
    for(int i=k; i<wageSize; i++){
        if(w[i].ind < qmax){
            //printf("%d %d\n",w[i].q,quals[w[i].ind].q);
            qsum += w[i].q - quals[qmax].q;
            quals[qmax].b = false;
            quals[w[i].ind].b = true;

            int j=qmax;
            //printf("%d %d %d\n",qmax,quals[qmax].q,qsum);
            while(!quals[j].b){
                j--;
            }
            qmax = j;

            cost = w[i].wpq*qsum;
            if(out>cost){
                out = cost;
            }
        }
    }
    
    printf("wpq:%f %d %f\n",wpq,qsum,cost);

    for(int i=0; i<wageSize; i++){
        printf("%d,%d,%f,%d\n",w[i].q,w[i].w,w[i].wpq,w[i].ind);
    }
    
    return out;
}
```
## Draft
```C
struct worker {
    int q;
    int w;
    double wpq;
    int flag;
};

int compareQuality(struct worker *a, struct worker *b){
    return a->q - b->q;
}

int compareWPQ(struct worker *a, struct worker *b){
    if(a->wpq > b->wpq){
        return 1;
    } else if(a->wpq == b->wpq){
        return 0;
    } else {
        return -1;
    }
}

int compareWage(struct worker *a, struct worker *b){
    return a->w - b->w;
}

int compareFlag(struct worker *a, struct worker *b){
    return a->flag - b->flag;
}

double mincostToHireWorkers(int* quality, int qualitySize, int* wage, int wageSize, int k) {
    struct worker *w;
    w = malloc(sizeof(struct worker)*wageSize);
    
    for(int i=0; i<wageSize; i++){
        w[i].q = quality[i];
        w[i].w = wage[i];
        w[i].wpq = (double)wage[i]/quality[i];
        w[i].flag = 0;
    }

    double out, wpq, cost;
    int qsum;

    out = 1E+37;

    // greedy quality
    qsort(w,wageSize,sizeof(struct worker),compareQuality);
    qsum = 0;
    wpq = 0.0;
    for(int i=0; i<k; i++){
        qsum += w[i].q;
        if(wpq < w[i].wpq){
            wpq = w[i].wpq;
        }
        w[i].flag += 1;
    }

    cost = wpq*qsum;
    if(out>cost){
        out = cost;
    }
    printf("wpq:%f %d %f\n",wpq,qsum,cost);

    // greedy wpg
    qsort(w,wageSize,sizeof(struct worker),compareWPQ);
    qsum = 0;
    for(int i=0; i<k; i++){
        qsum += w[i].q;
        w[i].flag += 2;
    }
    wpq = w[k-1].wpq;

    cost = wpq*qsum;
    if(out>cost){
        out = cost;
    }
    printf("wpq:%f %d %f\n",wpq,qsum,cost);

    // greedy wage
    qsort(w,wageSize,sizeof(struct worker),compareWage);
    qsum = 0;
    wpq = 0.0;
    for(int i=0; i<k; i++){
        qsum += w[i].q;
        if(wpq < w[i].wpq){
            wpq = w[i].wpq;
        }
    }

    cost = wpq*qsum;
    if(out>cost){
        out = cost;
    }
    printf("wpq:%f %d %f\n",wpq,qsum,cost);


    qsort(w,wageSize,sizeof(struct worker),compareFlag);
    for(int i=0; i<wageSize; i++){
        printf("%d,%d,%f,%d\n",w[i].q,w[i].w,w[i].wpq,w[i].flag);
    }

    return out;
}
```

# 2024-05-10
[786. K-th Smallest Prime Fraction](https://leetcode.com/problems/k-th-smallest-prime-fraction/)

## Slow and lazy but works
```C
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */

struct frac{
    int n;
    int d;
};

int compare(struct frac *a, struct frac *b){
    return a->n*b->d - b->n*a->d;
}

int* kthSmallestPrimeFraction(int* arr, int arrSize, int k, int* returnSize) {
    struct frac *f;
    int n, c;
    n = arrSize*(arrSize-1)/2;
    f = malloc(sizeof(struct frac)*n);
    c = 0;
    for(int i=0; i<arrSize; i++){
        for(int j=i+1; j<arrSize; j++){
            f[c].d = arr[j];
            f[c].n = arr[i];
            c++;
        }
    }

    qsort(f,n,sizeof(struct frac),compare);
    /*
    for(int i=0; i<n; i++){
        printf("%d/%d ",f[i].n,f[i].d);
    }
    */

    int *out;
    out = malloc(sizeof(int)*2);
    out[0] = f[k-1].n;
    out[1] = f[k-1].d;

    *returnSize = 2;
    return out;
}
```
# 2024-05-09
[3075. Maximize Happiness of Selected Children](https://leetcode.com/problems/maximize-happiness-of-selected-children/)
1 hour.
## Lazy but works
```C
int compare(const int *a, const int *b){
    return *b - *a;
}

long long maximumHappinessSum(int* happiness, int happinessSize, int k) {
    qsort(happiness, happinessSize, sizeof(int), compare);
    /*
    for(int i=0; i<happinessSize; i++){
        printf("%d ", happiness[i]);
    }
    */
    long long sum;
    sum = 0;
    for(int i=0; i<k; i++){
        int a;
        a = happiness[i] - i;
        if(a>0){
            sum += a;
        } else {
            break;
        }
    }

    return sum;
}
```
# 2024-05-08
[506. Relative Ranks](https://leetcode.com/problems/relative-ranks/)
3 hours.
Decently fast. Used a struct to keep things together, though uses a lot of memory.
Associate initial position with scores. Sort by score/rank. Associate rank. Sort by position. Done.
A lot of the work is just handling the strings. (The tedious part...)
Marked as "Easy" but implementation is very detailed...

Sorting once then simply look up position. Less memory since strings don't need to be associated.
## Sort Once (Better)
```C
struct node {
    int score;
    int pos;
};

int compareScore(struct node *a, struct node *b){
    return b->score - a->score;
}

char** findRelativeRanks(int* score, int scoreSize, int* returnSize) {
    struct node *ranks;
    ranks = malloc(sizeof(struct node) * scoreSize);

    for(int i=0; i<scoreSize; i++){
        ranks[i].score = score[i];
        ranks[i].pos = i;
    }

    qsort(ranks,scoreSize,sizeof(struct node),compareScore);

    char **s;
    s = malloc(sizeof(char*)*scoreSize);
    
    if(scoreSize==1){
        char *a; a = malloc(sizeof(char)*11);
        strcpy(a,"Gold Medal"); s[0]=a;
    } else if(scoreSize==2){
        char *a; a = malloc(sizeof(char)*24);
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a;
    } else if(scoreSize==3){
        char *a; a = malloc(sizeof(char)*47);
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a;
    } else if(scoreSize<9){
        char *a; a = malloc(sizeof(char)*(47+(scoreSize-3)*2));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<scoreSize; i++){
            sprintf(a,"%d",i+1); s[i]=a; a+=2;
        }
    } else if(scoreSize<99){
        char *a; a = malloc(sizeof(char)*(47+6*2+(scoreSize-9)*3));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<9; i++)        { sprintf(a,"%d",i+1); s[i]=a; a+=2; }
        for(int i=9; i<scoreSize; i++){ sprintf(a,"%d",i+1); s[i]=a; a+=3; }
    } else if(scoreSize<999){
        char *a; a = malloc(sizeof(char)*(47+6*2+90*3+(scoreSize-99)*4));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<9; i++)         { sprintf(a,"%d",i+1); s[i]=a; a+=2; }
        for(int i=9; i<99; i++)        { sprintf(a,"%d",i+1); s[i]=a; a+=3; }
        for(int i=99; i<scoreSize; i++){ sprintf(a,"%d",i+1); s[i]=a; a+=4; }
    } else if(scoreSize<9999){
        char *a; a = malloc(sizeof(char)*(47+6*2+90*3+900*4+(scoreSize-999)*5));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<9; i++)          { sprintf(a,"%d",i+1); s[i]=a; a+=2; }
        for(int i=9; i<99; i++)         { sprintf(a,"%d",i+1); s[i]=a; a+=3; }
        for(int i=99; i<999; i++)       { sprintf(a,"%d",i+1); s[i]=a; a+=4; }
        for(int i=999; i<scoreSize; i++){ sprintf(a,"%d",i+1); s[i]=a; a+=5; }
    } else if(scoreSize<99999){
        char *a; a = malloc(sizeof(char)*(47+6*2+90*3+900*4+9000*5+(scoreSize-9999)*6));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<9; i++)           { sprintf(a,"%d",i+1); s[i]=a; a+=2; }
        for(int i=9; i<99; i++)          { sprintf(a,"%d",i+1); s[i]=a; a+=3; }
        for(int i=99; i<999; i++)        { sprintf(a,"%d",i+1); s[i]=a; a+=4; }
        for(int i=999; i<9999; i++)      { sprintf(a,"%d",i+1); s[i]=a; a+=5; }
        for(int i=9999; i<scoreSize; i++){ sprintf(a,"%d",i+1); s[i]=a; a+=6; }
    }

    char **out;
    out = malloc(sizeof(char*)*scoreSize);
    for(int i=0; i<scoreSize; i++){
        out[ranks[i].pos] = s[i];
    }

    // free makes it slower!
    // free(ranks);
    *returnSize = scoreSize;
    return out;
}

```
## Sort Twice (Decently fast but uses a lot of memory.)
```C
struct node {
    int score;
    int pos;
    char *rank;
};

int compareScore(struct node *a, struct node *b){
    return b->score - a->score;
}

int comparePos(struct node *a, struct node *b){
    return a->pos - b->pos;
}

char** findRelativeRanks(int* score, int scoreSize, int* returnSize) {
    struct node *ranks;
    ranks = malloc(sizeof(struct node) * scoreSize);

    for(int i=0; i<scoreSize; i++){
        ranks[i].score = score[i];
        ranks[i].pos = i;
    }

    qsort(ranks,scoreSize,sizeof(struct node),compareScore);
    /*
    for(int i=0; i<scoreSize; i++){
        printf("%d %d\n",ranks[i].score,ranks[i].pos);
    }
    */

    char **s;
    s = malloc(sizeof(char*)*scoreSize);
    
    if(scoreSize==1){
        char *a; a = malloc(sizeof(char)*11);
        strcpy(a,"Gold Medal"); s[0]=a;
    } else if(scoreSize==2){
        char *a; a = malloc(sizeof(char)*24);
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a;
    } else if(scoreSize==3){
        char *a; a = malloc(sizeof(char)*47);
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a;
    } else if(scoreSize<9){
        char *a; a = malloc(sizeof(char)*(47+(scoreSize-3)*2));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<scoreSize; i++){
            sprintf(a,"%d",i+1); s[i]=a; a+=2;
        }
    } else if(scoreSize<99){
        char *a; a = malloc(sizeof(char)*(47+6*2+(scoreSize-9)*3));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<9; i++)        { sprintf(a,"%d",i+1); s[i]=a; a+=2; }
        for(int i=9; i<scoreSize; i++){ sprintf(a,"%d",i+1); s[i]=a; a+=3; }
    } else if(scoreSize<999){
        char *a; a = malloc(sizeof(char)*(47+6*2+90*3+(scoreSize-99)*4));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<9; i++)         { sprintf(a,"%d",i+1); s[i]=a; a+=2; }
        for(int i=9; i<99; i++)        { sprintf(a,"%d",i+1); s[i]=a; a+=3; }
        for(int i=99; i<scoreSize; i++){ sprintf(a,"%d",i+1); s[i]=a; a+=4; }
    } else if(scoreSize<9999){
        char *a; a = malloc(sizeof(char)*(47+6*2+90*3+900*4+(scoreSize-999)*5));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<9; i++)          { sprintf(a,"%d",i+1); s[i]=a; a+=2; }
        for(int i=9; i<99; i++)         { sprintf(a,"%d",i+1); s[i]=a; a+=3; }
        for(int i=99; i<999; i++)       { sprintf(a,"%d",i+1); s[i]=a; a+=4; }
        for(int i=999; i<scoreSize; i++){ sprintf(a,"%d",i+1); s[i]=a; a+=5; }
    } else if(scoreSize<99999){
        char *a; a = malloc(sizeof(char)*(47+6*2+90*3+900*4+9000*5+(scoreSize-9999)*6));
        strcpy(a,"Gold Medal"); s[0]=a; a+=11;
        strcpy(a,"Silver Medal"); s[1]=a; a+=13;
        strcpy(a,"Bronze Medal"); s[2]=a; a+=13;
        for(int i=3; i<9; i++)           { sprintf(a,"%d",i+1); s[i]=a; a+=2; }
        for(int i=9; i<99; i++)          { sprintf(a,"%d",i+1); s[i]=a; a+=3; }
        for(int i=99; i<999; i++)        { sprintf(a,"%d",i+1); s[i]=a; a+=4; }
        for(int i=999; i<9999; i++)      { sprintf(a,"%d",i+1); s[i]=a; a+=5; }
        for(int i=9999; i<scoreSize; i++){ sprintf(a,"%d",i+1); s[i]=a; a+=6; }
    }

    for(int i=0; i<scoreSize; i++){
        ranks[i].rank = s[i];
    }
    qsort(ranks,scoreSize,sizeof(struct node),comparePos);
    for(int i=0; i<scoreSize; i++){
        s[i] = ranks[i].rank;
    }

    // free makes it slower!
    // free(ranks);
    *returnSize = scoreSize;
    return s;
}

```
# 2024-05-07
[2816. Double a Number Represented as a Linked List](https://leetcode.com/problems/double-a-number-represented-as-a-linked-list/)
Easy.
Changing two lines makes it significantly faster. Maybe...
## Fast (Only few changes... Weird compiler shit.)
```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* doubleIt(struct ListNode* head){
    //struct ListNode *curr, *next, *out;
    struct ListNode *curr, *out;
    curr = head;

    if(curr->val > 4){
        struct ListNode *t;
        t = malloc(sizeof(struct ListNode));
        t->val = 0;
        t->next = curr;
        curr = t;
    }
    out = curr;

    while(curr->next){
        //next = curr->next;
        curr->val = ((curr->val*2)%10) + ((curr->next->val*2)/10);
        curr = curr->next;
    }
    curr->val = ((curr->val*2)%10);

    return out;
}
```
## Standard Approach
```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* doubleIt(struct ListNode* head){
    struct ListNode *curr, *next, *out;
    curr = head;

    if(curr->val > 4){
        struct ListNode *t;
        t = malloc(sizeof(struct ListNode));
        t->val = 0;
        t->next = curr;
        curr = t;
    }
    out = curr;

    while(curr->next){
        next = curr->next;
        curr->val = ((curr->val*2)%10) + ((next->val*2)/10);
        curr = next;
    }
    curr->val = ((curr->val*2)%10);

    return out;
}
```
# 2024-05-06
[2487. Remove Nodes From Linked List](https://leetcode.com/problems/remove-nodes-from-linked-list/)

## Simple
```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* removeNodes(struct ListNode* head) {
    if(head->next == NULL){
        return head;
    }

    // reverse list
    struct ListNode *curr, *prev, *next;

    prev = NULL;
    curr = head;
    while(curr->next){
        next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    curr->next = prev;

    // keep max nodes
    struct ListNode *max, *out;
    out = curr;
    max = curr;
    while(curr->next){
        curr = curr->next;
        if(curr->val >= max->val){
            max->next = curr;
            max = curr;
        }
    }
    max->next = NULL;

    // reverse
    prev = NULL;
    curr = out;
    while(curr->next){
        next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    curr->next = prev;
    out = curr;

    return out;
}
```
# 2024-05-05
[237. Delete Node in a Linked List](https://leetcode.com/problems/delete-node-in-a-linked-list/)
7 min.
## Easy
```C
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
void deleteNode(struct ListNode* node) {
    int v;
    struct ListNode *n;
    n = node->next;
    v = n->val;

    node->next = n->next;
    node->val = v;
}
```
# 2024-05-04

## Simple but slow (works)
```C
int compare(const void* a, const void* b){
  return (*(int*)b-*(int*)a);
}

int numRescueBoats(int* people, int peopleSize, int limit) {
    // sort high to low
    qsort(people,peopleSize,sizeof(int),compare);

    int count;
    count = 0;
    int s, e;
    s = 0;
    e = peopleSize-1;

    while(s<e){
        if(people[s]+people[e]<=limit){
            count++;
            s++;
            e--;
        }else{
            count++;
            s++;
        }
    }
    if(s==e){
        count++;
    }

    return count;
}
```
## First Idea (Doesn't work)
```C
int compare(const void* a, const void* b){
  return (*(int*)b-*(int*)a);
}

int numRescueBoats(int* people, int peopleSize, int limit) {
    // sort high to low
    qsort(people,peopleSize,sizeof(int),compare);

    /*
    for(int i=0; i<peopleSize; i++){
        printf("%d ",people[i]);
    }
    */

    int count, sum;
    count = 0;
    sum = 0;
    for(int i=0; i<peopleSize; i++){
        sum += people[i];
        if(sum > limit){
            sum -= people[i];
            
        }
    }

    return 0;
}
```

# 2024-05-03
[165. Compare Version Numbers](https://leetcode.com/problems/compare-version-numbers/)

## Basic
```C
int compareVersion(char* version1, char* version2) {
    /*
    int a,b;
    a = atoi(version1);
    b = atoi(version2);

    printf("%d %d\n",a,b);
    */
    char *a, *b;
    a = version1;
    b = version2;
    int c,d;
    c = atoi(a);
    d = atoi(b);

    while(c==d){
        while(a[0] && a[0]!='.'){
            a++;
        }
        while(b[0] && b[0]!='.'){
            b++;
        }

        if(a[0]==0 && b[0]==0){
            return 0;
        } else if(a[0] && b[0]) {
            a++;
            b++;
            c = atoi(a);
            d = atoi(b);
        } else if(a[0]==0) {
            b++;
            c = 0;
            d = atoi(b);
            if(d){
                return -1;
            }
        } else {
            a++;
            c = atoi(a);
            d = 0;
            if(c){
                return 1;
            }
        }
        
    }

    if(c<d){
        return -1;
    } else if(c>d) {
        return 1;
    } else {
        return 0;
    }
}
```
# 2024-05-02
[2441. Largest Positive Integer That Exists With Its Negative](https://leetcode.com/problems/largest-positive-integer-that-exists-with-its-negative/)
## Sort & Find
```C
int compare(const void* a, const void* b){
    return (*(int*)a-*(int*)b);
}

int findMaxK(int* nums, int numsSize) {
    qsort(nums,numsSize,sizeof(int),compare);
    int n, m, sum;
    n=0; m=numsSize-1;

    /*
    for(int i=0; i<numsSize; i++){
        printf("%d ",nums[i]);
    }
    */
    
    sum = nums[n]+nums[m];
    while(sum!=0 && n<m){
        if(sum<0){
            n++;
        }else{
            m--;
        }
        sum = nums[n]+nums[m];
    }

    if(sum){
        return -1;
    }else{
        return nums[m];
    }
}
```

# 2024-05-01
[2000. Reverse Prefix of Word](https://leetcode.com/problems/reverse-prefix-of-word/)
## Easy Peasy
```C
char* reversePrefix(char* word, char ch) {
    int s, e;
    s=0;
    e=0;
    while(word[e] && word[e]!=ch){
        e++;
    }
    if(word[e]){
        while(s<e){
            int t;
            t = word[e];
            word[e] = word[s];
            word[s] = t;
            s++;
            e--;
        }
    }
    return word;
}
```
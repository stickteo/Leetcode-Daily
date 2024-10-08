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

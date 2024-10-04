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

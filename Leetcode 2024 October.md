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

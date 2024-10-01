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

---
title: Big O Analysis
layout: blog
category: [Fundamentals]
excerpt: When it comes to Data Structures & Algorithms, there are 2 main factors that need to be analyzed every time. Time taken by the algorithm to finish [Time Complexity] Space needed by the algorithm for running the computation [Space Complexity] If your algorithm takes infinite amount of time to complete running, then your algorithm is...
---

When it comes to Data Structures & Algorithms, there are 2 main factors that need to be analyzed every time.

1. Time taken by the algorithm to finish **[Time Complexity]**
2. Space needed by the algorithm for running the computation **[Space Complexity]**

If your algorithm takes infinite amount of time to complete running, then your algorithm is of no use. Similarly, if your algorithm takes all the space in the internet for it to do its computation, then also your algorithm is of no use.

Therefore, a standardized way of calculating complexity is required. This standardized way is called “**Big-O Notation**“.

> Big-O Notation gives an upper bound of the complexity in the **worst case**, helping to quantity performance as the input size becomes **arbitrarily large**.

In Big-O, we refer to the size of the input given to the algorithm as “n”. The following are the Big-O complexities ordered from smallest to largest.

- Constant Time : O(1)
- Logarithmic Time : O(log(n))
- Linear Time : O(n)
- Linearithmic Time : O(nlog(n))
- Quadric Time : O(n<sup>2</sup>)
- Cubic Time : O(n<sup>3</sup>)
- Exponential Time : O(b<sup>n</sup>), where b>1
- Factorial Time : O(n!)

For example, if your algorithm takes constant time to finish, we say that your algorithm’s Time Complexity is O(1) [pronounced – Big O of 1].

## Big-O Properties

As we mentioned earlier, Big-O only cares about the worst case. So, we don’t care when “n” is small. We only care out “n” becoming large.

**Property 1: We can safely ignore constants while calculating the complexity**

So, **O(n + c) = O(n)** where c is some constant.

Similarly, **O(cn) = O(n)** where c is some constant > 0

Suppose f(n) = 8log(n)<sup>3</sup> + 12n<sup>2</sup> + 7n<sup>3</sup> + 5, we can say the O(f(n)) = **O(n<sup>3</sup>)** because when n grows, the most dominant factor that grows in this expression is n<sup>3</sup>.

## Examples of Big-O Analysis

#### CONSTANT TIME COMPLEXITY

Suppose we have an algorithm as shown below:

```
a = 1
b = 2
c = a + 4*b
```

We can easily say that the Big-O of this algorithm is O(1). The time complexity is constant because the input variables are just 2 variables (which is a & b) and the c is calculated only based on a & b. Here, the concept of “n” itself doesn’t occur due to which the algorithm runs in constant time.

#### LINEAR TIME COMPLEXITY

Suppose we have an algorithm as shown below:

```
i = 0
while i < n
    i = i + 1
```

Since the while loop runs for “i” until it reaches “n”, we can say that the algorithm takes O(n) to complete its computation.

#### QUADRIC TIME COMPLEXITY

Suppose we have an algorithm as shown below:

```
for (i = 0; i < n; i = i + 1)
    for (j = 0; j < n; j = j + 1)
```

Here, we can see that two loops execute “n” times each within itself. That means when i = 0 in the first loop, the inner loop of j executes “n” times. Similarly, when “i” becomes 1, “j” loop executes “n” times and so on. Therefore, the time complexity of this algorithm is O(n<sup>2</sup>).

Let’s make a small twist to the previous algorithm as shown below: **[we replaced j = 0 in the inner loop to j = 1]**

```
for (i = 0; i < n; i = i + 1)
    for (j = i; j < n; j = j + 1)
```

Here, we will see that the algorithm executes for different values of i & j as described below.

| value of i | number of times j loop executes |
| ---------- | ------------------------------- |
| 0          | n                               |
| 1          | n-1                             |
| 2          | n-2                             |
| 3          | n-3                             |
| ...        | ...                             |
| n-3        | 3                               |
| n-2        | 2                               |
| n-1        | 1                               |
| n          | 0                               |

Therefore, the total number of looping done when i changes its value from 0 to n = (n) + (n-1) + (n-2) + (n-3) + … + 3 + 2 + 1
This is a familiar sequence in mathematics and equates to n(n+1)/2
So, the time complexity of this algorithm is O(n(n+1)/2) = O(n<sup>2</sup>/2 + n/2) = **O(n<sup>2</sup>)**

#### LOGARITHMIC TIME COMPLEXITY

Suppose we have an algorithm as shown below:

- the “array” is in sorted form
- “low” represents the index of the first item in the array
- “high” represents the index of the last item in the array
- “mid” represents the index of the middle item in the array
- “value” represents a value that we want to find in this array

```
low = 0
high = n-1
while low <= high
    mid = (low + high) / 2
    if array[mid] == value
        return mid
    else if array[mid] < value
        low = mid + 1
    else if array[mid] > value
        high = mid - 1
```

This algorithm (popularly known as Binary Search) divides the array into 2 halves (low to mid & mid to high) and checks if the value to search is less than or greater than mid. If the value is less than mid, the search is done on the left side portion (low to mid). If the value is greater than mid, the search is done only on the right side portion (mid to high). This strategy of divide & conquer reduces the search complexity to logarithmic level. Therefore, the time complexity of this algorithm becomes **O(log(n))**.

![Binary Search](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/binary-search.png)

Likewise (for your knowledge),

- **Finding all subsets of a set takes O(2<sup>n</sup>)**
- **Finding all permutations of a string takes O(n!)**
- **Sorting using mergesort takes O(nlog(n))**
- **Iterating over all the cells in a matrix of size m\*n takes O(mn)**

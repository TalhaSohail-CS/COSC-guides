Dynamic programming is an algorithmic technique where the problem is broken down into subproblems whose results are saved in order to help us solve the overall problem.
There are two types of Dynamic Programming: 
- Memoization: Recursive
- Tabulation: Iterative

### Memoization:
- Top-Down approach
- Check if the value has already been calculated
- If so, output it, otherwise, calculate and store in a lookup table
- Save the solutions as you approach your base case
- Start from the final problem, recursively compute the subproblems when needed

Memoization Recipe:

1. Make it work
- Visualize the problem as a tree
- Implement the tree using recursion
- Test it

2. Make it efficient
- Add a memo object
- Add a base case to return memo values
- Store return values into the memo

### Tabulation
- Bottom-Up approach
- Initialize a table for lookup
- Determine the execution order of your algorithm (e.g. row by row)
- Fill up the table in the execution order
- Start from 1, cache each of the results and continue to iterate until we reach n
- Build towards the solution as you continually compute each subresult

Tabulation Recipe:

- visualize the problem as a table
- size the table based on the inputs
- initialize the table with default values
- seed the trivial answer into the table
- iterate through the table
- fill further positions based on the current position

3 steps to DP
1. Recursion
2. Store (Memoize)
3. Bottom-Up (Tabulation)

# Fibonacci

## Naive Solution (Time Complexity = O(2^n))
```py
function fibonacci(n)
  if n <= 2
    result = 1
  else
    result = fibonacci(n-1) + fibonacci(n-2)
```
What's the issue with this code?
We have a lot of repeated calls. fib(2) is called 3 times if n = 5, fib(1) and fib(3) are called twice, etc.
Fibonacci Recursive is O(2^n) since you are making 2 recursive calls everytime.

With Dynamic Programming you can just store the results in an array and access the list whenever you need a result. This makes the code O(n) which is more efficient

Visualization for Fibonacci:
fill an array of n size with all -1
we don't know what fib(n) is so solve for fib(n-1)
we don't know what fib(n-1) is so solve for fib(n-2)... fib(n-n)
we know what fib(0) is, now we can store it and solve fib(1)...fib(n)
since we have fib(0) stored we never have to call fib(0) again, we can just check if its in the array
same for fib(1) once we solve it, fib(2), fib(3)...fib(n)

### Pseudocode

Dynamic Programming (Memoization)
```py
function fibonacci(n)
  if n not in memo:
    memo[n] = fibonacci(n - 1) + fibonacci(n-2)
  return memo[n]
```

Dynamic Programming (Tabulation)
```py
function fibonacci(n)
  lookup_table[0, 1, 0, 0, ..., n, n+1] # length n + 1 table
  for i from 2 to n:
    lookup_table[i] = lookup_table[i - 1] + lookup_table[i - 2]
  return lookup_table[n]
```

### Python Code

```py
def fib(n, memo): # Memoization
  if memo[n] != null:
    return memo[n]
  if n == 1 or n == 2:
    result = 1
  else:
    result = fib(n-1) + fib(n-2)
    memo[n] = result
    return result

def fib_bottom_up(n): # Tabulation
  if n == 1 or n == 2:
    return 1
  bottom_up = new int[n+1]
  bottom_up[1] = 1
  bottom_up[2] = 1
  for i from 3 up to n:
    bottom_up[i] = bottom_up[i-1] + bottom_up[i-2]
  return bottom_up[n]
```

### Visualization

```js
//Naive Solution
const fib = (n) => {
  if (n <= 2) return 1;
  return fib(n - 1) + fib(n - 2);
};
```
```
Visualization of Naive Fibonacci
E.G. fib(6) for an array 0 - 6 inclusive
                6
              /   \
             5     4
            / \   /  \
           4   3  3   2
          /\  /\  /\  /\
         3 2 2 1 2 1 1  0

Notice how 3, 2, and 1 are called more than once.
Why not just store it so we don't have to keep calling it?
```
```js
//Memoized Solution
const fib = (n, memo = {}) => {
  if (n in memo) return memo[n];
  if (n <= 2) return 1;
  memo[n] = fib(n-1, memo) + fib(n-2, memo);
  return memo[n];
};

//Tabulation Solution
const fib = (b) => {
  const table = Array(n + 1).fill(0);
  table[1] = 1;
  for (let i = 0; i <= n; i++) {
    table[i + 1] += table[i];
    table[i + 2] += table[i];
  }
  return table[n];
}
```
```
Visualization of Memoization DP Fibonacci
E.G. fib(6) for an array 0 - 6 inclusive
[0, 1, 0, 0, 0, 0, 0]
            6
           /
          5
         /
        4
       /
      3
     /
    2
   /
  1 
 /
0

Visualization of Tabulation DP Fibonacci
E.G. fib(6) for an array 0 - 6 inclusive
[0, 1, 0, 0, 0, 0, 0]
Pass 0: we know fib(0) is 0, so add 0 (the value of fib(0)) to fib(1) and fib(2)
[0, 1, 0, 0, 0, 0, 0]
Pass 1: we know fib(1) is 1, so add 1 (the value of fib(1)) to fib(2) and fib(3)
[0, 1, 1, 1, 0, 0, 0]
Pass 2: we know fib(2) since we know fib(1) and fib(0), so we can add 1 (the value of fib(2)) to fib(3) and fib(4)
[0, 1, 1, 2, 1, 0, 0]
Pass 3: we know fib(3) since we know fib(2) and fib(1), so we can add 2 (the value of fib(3)) to fib(4) and fib(5)
[0, 1, 1, 2, 3, 2, 0]
Pass 4: we know fib(4) since we know fib(3) and fib(2), so we can add 3 (the value of fib(4)) to fib(5) and fib(6)
[0, 1, 1, 2, 3, 5, 3]
Pass 5: we know fib(5) since we know fib(4) and fib(3) so we can add 5 (the value of fib(5)) to fib(6)
[0, 1, 1, 2, 3, 5, 8]
```

# gridTraveler

Write a function gridTraveler, that given a m*n matrix, count how many ways can you traverse from the top left to the bottom right, given you can only move down or right

Naive Solution Visualization:
```
     0,0
    /   \
   D     R
  / \   / \
 D   R D   R

note the # of times we call D/R
```
Naive Solution:
```js
const gridTraveler = (m, n) => {
  if (m === 1 && n == 1) return 1;
  if (m === 0 || n === 0) return 0;
  return gridTraveler(m - 1, n) + gridTraveler(m, n - 1);
};
```
Memoization Solution:
```js
// The memoization visualization is just the starting point because all D and Rs are duplicates lol
const gridTraveler = (m, n, memo={}) => {
  const key = m + ',' + n;
  if (key in memo) return memo[key];
  if(m === 1 && n == 1) return 1;
  if(m === 0 || n === 0) return 0;
  memo[key] = gridTraveler(m - 1, n, memo) + gridTraveler(m , n - 1. memo);
  return memo[key];
```
Tabulation Visualization:
```
3x3 matrix, 6 ways to traverse
[0, 0, 0, 0]
[0, 0, 0, 0]
[0, 0, 0, 0]
[0, 0, 0, 0]

Why 4x4? so our coordinates match up properly (0 - 3 column, row)

Pass 0: We know in a 1 by 1 grid, there is only 1 way to travel through it, so mark (1, 1) as 1
[0, 0, 0, 0]
[0, 1, 0, 0]
[0, 0, 0, 0]
[0, 0, 0, 0]

Pass 1: since we filled out that spot at 1x1, we can add 1 to the spots below and to its right, like so:
[0, 0, 0, 0]
[0, 1, 1, 0]
[0, 1, 0, 0]
[0, 0, 0, 0]

Pass 2 - n: Repeat

[0, 0, 0, 0]
[0, 1, 1, 1]
[0, 1, 2, 0]
[0, 1, 0, 0]
...
[0, 0, 0, 0]
[0, 1, 1, 1]
[0, 1, 2, 3]
[0, 1, 3, 6]
```
```js
const gridTraveler = (m, n) => {
  const table = Array(m + 1)
    .fill()
    .map(() => Array(n + 1).fill(0));

  table[1][1] = 1

  for (let i = 0; i <= m; i++) {
    for (let j = 0; j M= n; j++) {
      const current = table[i][j];
      if(j + 1 <= n) table[i][j + 1] += current;
      if(i + 1 <= m) table[i + 1][j] += current;
    }
  }

  return table[m][n];
};
```

# Coin Problem: Minimum Bills
In a strange country (Algostan), the currency is available in the following denominations: $1, $4, $7, $13, $28, $52, $91, $365. Find the minimum bills that add up to a given sum $k.
```py
function change(k)
  if k < 0:
    return infinity
  if k not in memo:
    lookup[k] = infinity
    for coin in coins:
      lookup[k] = min(change(k - coin), k)+1
  return lookup[k]
```

# Minimum Palindromes
Find the minimum number of palindromes that make up the given string

Input format:
first line is a single number n
the following line consists of length n string (character array) that is the string A[1..n]

### Bottom Up Solution (Tabulation)
1. We can approach this problem by creating a lookup table that is the a n x n matrix, where n is the length of the strings
2. We’ll also keep track of the number of cuts made with an n length array.
3. Iterate over the string, choosing a starting index and ending index, essentially checking each substring
4. Check if the substring is a palindrome and marking any palindromes at table[start][end]
5. Calculate the minimum cuts and store into the array.

### Pseudocode

Declare two arrays:
1. Lookup table of boolean
2. Array of the number of cuts

Nested for loop to iterate over the different substrings.
Check if the characters match one another.

```py
function minimum_palindromes(length, string):
  if length == 0:
    return 0
  table = n x n array where all elements are false
  cuts = n length array initiliazed to [0,...0]
  for start in length:
    minCut = MAX
    for end in start+1:
      bool do_edges_match = string[start] equals string[end]
    bool is_start_inbounds = start <= end+1
    if do_edges_match and (is_start_inbounds or table[start-1]end[+1]):
      table[start][end] = true
      mincut = min(mincut, 0 if end is 0 else 1 + cuts[end-1])
    cuts[start] = mincut
  return cuts[length-1]+1
```

### Top Down Pseudocode (Memoization)
```py
function minimum_palindromes(length, string):
  # assume there is a local function called is_palindrome(start, end)
  function min_cut(i):
  if <= 0:
    return True
  else:
    return minimum of min_cut(j) + 1 for j in i if palindrome(j, i)

  return min_cut(string.length)
```

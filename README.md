In the above code, we take an adjacency matrix, a `k` value, and a given matrix size `n` and derive an almost equal color sequence if one exists.

1. We must check to see if the matrix is valid and do other constant time operations:
```python
# Check if the matrix is a valid adjacency matrix
  if n != len(adj_matrix[0]):
      print("Invalid adjacency matrix.")
      return

  # Initialize the color sequence
  color_sequence = [0] * n
```
  Both of these statements are constant time operations. According to [Python Wiki](https://wiki.python.org/moin/TimeComplexity), the `len()` function takes constant time because of the way the number is tracked. Therefore, `O(1)` complexity. The second statement is basically constant time because even with a huge matrix, it still wouldn't take too long. However, because it does go through the size of the matrix, it is `O(n)` complexity. Therefore, the first part of the algorithm is `O(n) + O(1) = O(n)` complexity.

2. Sort our nodes by their degrees. The number at `nodes_by_degree[x]` is the key of the vertex. According to [Wikipedia](https://en.wikipedia.org/wiki/Timsort), Python uses the Timsort algorithm in its standard library.
>Timsort is a hybrid, stable sorting algorithm, derived from merge sort and insertion sort, designed to perform well on many kinds of real-world data. It was implemented by Tim Peters in 2002 for use in the Python programming language. The algorithm finds subsequences of the data that are already ordered (runs) and uses them to sort the remainder more efficiently. This is done by merging runs until certain criteria are fulfilled. Timsort has been Python's standard sorting algorithm since version 2.3.
  
  ```python
  # Sort nodes by degree (number of connections) in descending order
  nodes_by_degree = sorted(range(n), key=lambda x: np.count_nonzero(adj_matrix[x]), reverse=True)
  ```

  The complexity of this algorithm is `O(nlogn)`, according to [Wikipedia](https://en.wikipedia.org/wiki/Timsort), which cites from this [scholary source](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/patsort-sigmod14.pdf)

  Therefore, the sorting step of this algorithm is `O(nlogn)` complexity.

3. We assign our colors greedily. To do so, we use the below code. Now, lets think about analyzing it. From the top, we are using simple undirected graphs, which means that there are no cycles. Lets use this information to analyze our greedy algorithm.

  a. Line 1 iterates through all the nodes sorted by their degree. This will ALWAYS take O(n) complexity, where `n` represents the number of nodes in the graph.

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 1    | `for node in nodes_by_degree:` | O(n) | O(n) | O(n)

  b. Line 3 declares an empty array for the neighbors to reside in. This is constant time no matter what.

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 3    |     `neighbors = []` | O(1) | O(1) | O(1)

  c. Line 5 iterates from `i` to `n` where `n` is the size of the matrix, or the number of nodes in the graph. Therefore, in all cases it is `O(n)` complexity where `n` is the size. However, this happens for EVERY node, so we make sure when we consider this in our final growth function, we multiply instead of adding.

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 5    |     `for i in range(n):` | O(n) | O(n) | O(n)

  d. Line 6 checks every time the loop runs if its next neighbor is equal to 1. In all cases, it runs everytime the loop does, or `n` times.

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 6    |         `if adj_matrix[node][i] == 1:` | O(n) | O(n) | O(n)

  e. IF line 7 gets reached, THEN we append a new neighbor. This is constant time in Python. However, we really have to pay attention to our cases here. In the worst case, this node is connected to EVERY other node minus itself, or `n-1` nodes. Therefore, this may happen `n-1` times in the worst case. On average, we can determine a so called, graph density, and use that to determine the probability that any given node has a certain number of edges. However you slice it though, it will still give O(n) time for this in the average case. The best case is that there are no neighbors so we don't have to worry, which would make this run ZERO times! Therefore, the runtime analysis of this line is as follows:

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 7    |             `neighbors.append(i)` |  O(n) | O(n) | 0

**At this point in the code, we have stepped out of the inner loop. We are now back in the outer loop.**

  f. Line 9 and 10 use a feature of Python called comprehension to build a generator that will be used to get a set of unique colors from the neighbors array built in the inner loop. This comprehension iterates through each element in neighbors array. Construction of this set in the worst case iterates through every neighbor but itself (no cycles), and on the average case would still give O(n) time to construct this set. In the worst case, there were no neighbors and an empty set will be returned. Therefore,

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 9    |     `neighbor_colors_generator = (color_sequence[n] for n in neighbors)` | O(n) | O(n) | O(1)
| 10   |     `neighbor_colors = set(neighbor_colors_generator)` |  O(n) | O(n) | O(1)

  g. Line 12 is responsible for creating the available colors set. It may seem like a linear operation, but in fact it is limited by the small number of colors to be chosen. Therefore, we can say that the runtime of this operation is O(k) in all cases

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 12   |     `available_colors = set(range(k)) - neighbor_colors` | O(k) | O(k) | O(k)

  h. Line 14 is responsible for checking if there is a set of available colors for the given node, and does so for each node. Therefore, it runs O(n) times in all cases.

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 14   |     `if available_colors:` | O(n) | O(n) | O(n)

  i. Line 15 assigns a color sequence of the node from the available colors. Using the min operation on the available colors is also an `O(k)` operation because min will at MOST search through `k` colors. However, not all cases are the same. In the worst case, each node has a color sequence found and it will have to search for the min. On average, we would like to assume that our algorithm is finding a color sequence for a node, so therefore it also is assumed to have to search for the min. In the "best case", there is no available colors and it will continue to the `else` block.

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 15   |         `color_sequence[node] = min(available_colors)` | O(k) | O(k) | 0

  j. Lines 16 through 18 ideally will NOT be ran in the worst or average cases, but will be ran in the worst case. Line 16 in the best case will be ran O(1) times because after this, the function exits and returns that no sequence exists. Therefore, the loop does not complete its whole iteration through `n` nodes. Regardless of where this happens in the iteration, the else statement will only be ran once. Therefore, it is an O(1) statement in all cases. The next lines 17 and 18 are both simple O(1) statements.

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 16   |     `else:` | 0 | 0 | O(1)
| 17   |         `print("No such a sequence exists.")` | 0 | 0 | O(1)
| 18   |         `return` | 0 | 0 | O(1)

  k. Finally, we return the array which is a simple O(1) statement in all cases.

| Line | Code | Worst Case | Average Case | Best Case
|------|------|-----|-----|-----|
| 20   | `return color_sequence` | O(1) | O(1) | O(1)

  l. Putting ALL of these complexities together, we can determine the complexity of the whole function. Our lines outside of the loop count which are the checking valid matrix, initializing a result array, sorting nodes by their degree, and finally the return statement count for:

  ```
  O(1) + O(n) + O(nlogn) + O(1) = O(nlogn) + O(n) + O(2)
  ```

  The loop is a little more complex to calculate though. The growth function calculated plus the complexities above gives us a result of:

  ```
  g(n,k) = 3n^2+nlogn+5n+2k+4
  ```

  However, due to the small amount that `k` represents, we can shrug it off as a constant time operation, giving us a growth function in terms of `n`

  ```
  g(n) = 3n^2+nlogn+5n+6
  ```

  This of course is a quadratic runtime, and can be shown to be Θ(n^2) via the limit definition.

  ```
  lim_x_to_inf(g(n)) = d, 0 < d < inf
  ```

  Evaluating the limit is too much to put into a markdown document, however doing so gives you a result of `3` which is enough to satisfy this algorithm into Θ(n^2) complexity.
# CMPS 2200  Recitation 03

**Name (Team Member 1):**___Maeren Hay______________________  
**Name (Team Member 2):**_________________________



## Analyzing a recursive, parallel algorithm


You were recently hired by Netflix to work on their movie recommendation
algorithm. A key part of the algorithm works by comparing two users'
movie ratings to determine how similar the users are. For example, to
find users similar to Mary, we start by having Mary rank all her movies.
Then, for another user Joe, we look at Joe's rankings and count how
often his pairwise rankings disagree with Mary's:

|      | Beetlejuice | Batman | Jackie Brown | Mr. Mom | Multiplicity |
| ---- | ----------- | ------ | ------------ | ------- | ------------ |
| Mary | 1           | 2      | 3            | 4       | 5            |
| Joe  | 1           | **3**  | **4**        | **2**   | 5            |

Here, Joe (somehow) liked *Mr. Mom* more than *Batman* and *Jackie
Brown*, so the number of disagreements is 2:
(3 <->  2, 4 <-> 2). More formally, a
disagreement occurs for indices (i,j) when (j > i) and
(value[j] < value[i]).

When you arrived at Netflix, you were shocked (shocked!) to see that
they were using this O(n^2) algorithm to solve the problem:



``` python
def num_disagreements_slow(ranks):
    """
    Params:
      ranks...list of ints for a user's move rankings (e.g., Joe in the example above)
    Returns:
      number of pairwise disagreements
    """
    count = 0
    for i, vi in enumerate(ranks):
        for j, vj in enumerate(ranks[i:]):
            if vj < vi:
                count += 1
    return count
```

``` python 
>>> num_disagreements_slow([1,3,4,2,5])
2
```

Armed with your CMPS 2200 knowledge, you quickly threw together this
recursive algorithm that you claim is both more efficient and easier to
run on the giant parallel processing cluster Netflix has.

``` python
def num_disagreements_fast(ranks):
    # base cases
    if len(ranks) <= 1:
        return (0, ranks)
    elif len(ranks) == 2:
        if ranks[1] < ranks[0]:
            return (1, [ranks[1], ranks[0]])  # found a disagreement
        else:
            return (0, ranks)
    # recursion
    else:
        left_disagreements, left_ranks = num_disagreements_fast(ranks[:len(ranks)//2])
        right_disagreements, right_ranks = num_disagreements_fast(ranks[len(ranks)//2:])
        
        combined_disagreements, combined_ranks = combine(left_ranks, right_ranks)

        return (left_disagreements + right_disagreements + combined_disagreements,
                combined_ranks)

def combine(left_ranks, right_ranks):
    i = j = 0
    result = []
    n_disagreements = 0
    while i < len(left_ranks) and j < len(right_ranks):
        if right_ranks[j] < left_ranks[i]: 
            n_disagreements += len(left_ranks[i:])   # found some disagreements
            result.append(right_ranks[j])
            j += 1
        else:
            result.append(left_ranks[i])
            i += 1
    
    result.extend(left_ranks[i:])
    result.extend(right_ranks[j:])
    print('combine: input=(%s, %s) returns=(%s, %s)' % 
          (left_ranks, right_ranks, n_disagreements, result))
    return n_disagreements, result

```

```python
>>> num_disagreements_fast([1,3,4,2,5])
combine: input=([4], [2, 5]) returns=(1, [2, 4, 5])
combine: input=([1, 3], [2, 4, 5]) returns=(1, [1, 2, 3, 4, 5])
(2, [1, 2, 3, 4, 5])
```

As so often happens, your boss demands theoretical proof that this will
be faster than their existing algorithm. To do so, complete the
following:

a) Describe, in your own words, what the `combine` method is doing and
what it returns.

.  
.  The combine method merges two sorted lists of movie rankings. To represent a disagreement, the method 
.  counts how many times an element from the right list is smaller than an element from the left list. 
.  It returns the total number of disagreements and the fully merged sorted list.
.  The combine method is similar to a merge sort, but with extra steps to count disagreements when elements are 
.  out of order. 
.  
.  
.  

b) Write the work recurrence formula for `num_disagreements_fast`. Please explain how do you have this.

.  W(n) = 2W(n/2) + O(n)
.  
.  the function splits the input into 2 halves (each with size n/2)
.  there are 2 recursive calls on size n/2 -> 2W (n/2)
.  the combine step does O(n) work, so we add + O(n)
.  

c) Solve this recurrence using any method you like. Please explain how do you have this.

.  
.  (using tree method)
.  level 0 : work = cn
.  level 1 : there are 2 calls of size (n/2), so total work= 2 * c(n/2) = cn
.  level 2 : there are 4 calls of size (n/4), so total work = 4 * c(n/4) = cn 
.  ...
.  level log n : (base case). There is constant work per call, but with n calls of size 1. Total work = cn
.  every level does cn amount of work, there are log n + 1 levels 
.  so, total work is W(n) = cn * (logn + 1) = O(nlogn)
.  
.  total work = O(nlogn)
.  


d) Assuming that your recursive calls to `num_disagreements_fast` are
done in parallel, write the span recurrence for your algorithm. Please explain how do you have this.

.  
.  If the two recursive calls run in parallel, then we only have to wait for one (the slower one)
.  so we pay for 1 recursive call --> S(n/2)
.  the combine step however takes O(n) time 
.  so the recurrence is S(n) = S(n/2) + O(n)
.  
.  
.  
.  
.  
.  
.  

e) Solve this recurrence using any method you like. Please explain how do you have this.

.  S(n) = S(n/2) + O(n)
.  
.  using recursion tree method:
.  each level adds a cost of cn, c(n/2), c(n/4),...
.  total span: S(n) = cn + c(n/2) + c(n/4) +...+ c*1
.  
.  this is a geometric series so:
.  S(n) = c * n * ( 1 + 1/2 + 1/4 + ...) = c * n * (2) = 2cn
.  
.  so the span is O(n)
.  
.  

f) If `ranks` is a list of size n, Netflix says it will give you
lg(n) processors to run your algorithm in parallel. What is the
upper bound on the runtime of this parallel implementation? (Hint: assume a Greedy
Scheduler). Please explain how do you have this.

W(n) = O(nlogn)
S(n) = O(n)
P = log n processors
plugging in :

T(n) <= nlogn/logn + n = n + n = 2n

so the upper bound on the parallel runtime is O(n)
(this is an improvement from the origninal O(n^2)!!)
 

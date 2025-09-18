# CMPS 2200  Recitation 04

**Name (Team Member 1):** Natalie Gockerman  
**Name (Team Member 2):**_________________________


In this lab you will practice using the `map` and `reduce` functions. These functions are commonly used together in a `map-reduce` framework, used by Google and others to parallelize and scale common computations.


## Part 1: Counting Words

In the first part, we will use map-reduce to count how often each word appears in a sequence of documents. E.g. if the input is two documents:

```python
['i am sam i am', 'sam is ham']
```

then the output should be

```python
[('am', 2), ('ham', 1), ('i', 2), ('is', 1), ('sam', 2)]
```

We have given you the implementation of the main map-reduce logic
```python
def run_map_reduce(map_f, reduce_f, docs)
```

To use this function to count words, you'll need to implement your own `map_f` and `reduce_f` functions, described below.

1. Complete `word_count_map` and test it with `test_word_count_map`. Please use doc.split() to split a string. 

2. Complete `word_count_reduce` and test it with `test_word_count_reduce`.

3. If the above are correct, then you should now be able to test it the full solution `test_word_count`

4. Assume that a word `w` appears `n` times. What is the **work** and **span** of `word_count_reduce` for this word, assuming a parallel implementation of the `reduce` function?
   
   We know word_count_reduce calls reduce(plus, 0, ones) on a list of n ones. The reduce() function splits the list in half recursively and combines the results with 1 plus.
   Examining the tree of the reduce function level 1 has n/2 additions, level 2 has n/4 additions, all the way to the root, which does 1 addition.
   Summing all levels (n/2) + (n/4) + ... + 1 = n - 1, so W(n) = n - 1 = Θ(n).
   To calculate the span, we know levels may compute paralell, but each must occure in order.
   The height of the tree = log2 n and each level completes one dependent addition, so S(n) = log2 n = Θ(log n).


6. Why are we going through all this trouble? Couldn't I just use this function to count words?

```python
docs = ['i am sam i am', 'sam is ham']
counts = {}
for doc in docs:
    for term in doc.split():
        counts[term] = counts.get(term, 0) + 1
# counts = {'i': 2, 'am': 2, 'sam': 2, 'is': 1, 'ham': 1}
```

What is the problem that prevents us from easily parallelizing this solution?

    This solution cannot be easily parallelized because of the "race condition", meaning that the 2 parallel runs will be racing to update the same number periodically (instead of seperately counting and summing together at the end), causing them to have errors and work over eachother.
    For example, if both parallel runs are counting and both find a matching term, they will both find that term and adds 1 to the orginial count. If this occurs at the same time, they will both see a value (assume 1), each will add 1 to that and reset the count to 2 when in reality it should be 3.
    MapReduce solves this issue because the parallelized version pairs independently of eachother, summing all matches in their side, then sum together at the end.


## Part 2: Sentiment analysis

Finally, we'll adapt our approach above to perform a simple type of sentiment analysis. Given a document, rather than counting words, we will instead count the number of positive and negative terms in the document, given a predefined list of terms. E.g., if the input sentence is `it was a terrible waste of time` and the terms `terrible` and `waste` are in our list of negative terms, then the output is

`[('negative', 1), ('negative', 1)]`

6. Complete the `sentiment_map` function to implement the above idea and test it with `test_sentiment_map`.

7. Since the output here is similar to the word count problem, we will reuse `word_count_reduce` to compute the total number of positive and negative terms in a sequence of documents. Confirm your results work by running `test_sentiment`.

---
title: "[Article] Revist 'iterator' in Depth"
date: 2022-12-22T16:46:49+08:00
draft: false
---

## Introduction
I'm using Google Bigquery in Python and find that the library uses iterator a lot. I'm curious about why they design the library in that way. To have a better unstanding, I revisit the concepts of iteratable and iterator. This article will show you
- the difference between iterator and iteratale,
- how the for loop works differently on iterator and iteratale.


## What is the difference between iterator and iteratale?
I take the following quote from [Python's wiki](https://wiki.python.org/moin/Iterator):

> 1. An iterable object is an object that implements `__iter__`, which is expected to return an iterator object.
> 2. An iterator object implements `__next__`, which is expected to return the next element of the iterable object that returned it, and to raise a StopIteration exception when no more elements are available.
> 3. In the simplest case, the iterable will implement `__next__` itself and return self in `__iter__`. 

However, this is not clear. It's easier to unstand with the following snippet:

```python
iteratable_list = [1, 2, 3]

# It echos to the 1st point from wiki: a list is an iterable and it returns an iterator by calling __iter__.
iterator_list = iter(iteratable_list) 

# It echos to the 2nd point: an iterator implements __next__.
print("The next element of iterator_list:", next(iterator_list))  

# It echos to the 3rd point: a simplest iterator will return itself by calling __iter__
iterator_itself = iter(iterator_list) 

print("The next element of iterator_itself:", next(iterator_itself))
print("Is iterator_list the same object as iterator_itself:", iterator_list is iterator_itself)
print("Is iterator_list the same object as iteratable_list:", iterator_list is iteratable_list)

```
Below is the output:
```python
The next element of iterator_list: 1
The next element of iterator_itself: 2
iterator_list is the same object as iterator_itself: True
terator_list is the same object as iteratable_list: False
```

For short, **iterator is one kind of iterable**. Below are the details:
- A list is an iterable, but not an iterator
- An iterator is an iterator, and it's also an interable because it has the `__iter__` method.
- List and iterator have different `__iter__` implementations: `iterator.__iter__()` returns itself but `list.__iter__()` does not.

## Coding with iterator
The fundamental method of an iterator is the `next()`. All other methods are built upon it.

### next
The `next()` method is already illusrated in the above section. Just be careful that it will raise an `StopIteration` when if no more element left:

```python
# expect to return 3
print("The next element of iterator_itself:", next(iterator_itself)) 

 # expect to get StopIteration because no element is left
print("The next element of iterator_itself:", next(iterator_itself))
```
Output:
```python
The next element of iterator_itself: 3
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
Cell In[3], line 2
      1 print("The next element of iterator_itself:", next(iterator_itself))
----> 2 print("The next element of iterator_itself:", next(iterator_itself))

StopIteration: 
```

### for loop
The `for` loop in Python requires the variable to be an iterable. We can use the loop on both list and iterator because both have `__iter__` implementation. However, the behaviors are very distinct if you want to loop them more than once.

```python
iteratable_list = [1, 2, 3]
iterator_list = iter(iteratable_list)

print("List 1st time iterating")
for i in iteratable_list:
    print(i)
print("End iterating")

print("======")


print("List 2nd time iterating")
for i in iteratable_list:
    print(i)
print("End iterating")

print("======")

print("Iterator 1st time iterating")
for i in iterator_list:
    print(i)
print("End iterating")with-bigquery-result-1
    print(i)
print("End iterating")

print("======")

```
Output:
```python
List 1st time iterating on list
1
2
3
End iterating
======
List 2nd time iterating
1
2
3
End iterating
======
Iterator 1st time iterating on list
1
2
3
End iterating
======
Iterator 2nd time iterating
End iterating
======
```

According to the above snippet and its ouput, you will find that:
- You can loop a list mutiple times with the same results.
- You get NOTHING if you loop an iterator more than once.

**But why?**

This is because `for` **doesn't loop over the iteratable** directly. Instead, **it loop over the iterator** returned by the `__iter__` method. Let me show you the code for this point. The for loop above is equvilant to the following code:
```python
iterable = [1, 2, 3]

# `for i in iterable:`
#     print(i)
# is equvilant to:
iterator = iter(iterable)
while True:
    try:
        i = next(iterator)
        print(i)
    except StopIteration:
        break
```

The `for` loop calls `iter(iteratable)` at the begining of the loop:
- When **list** is the iteratable, `next(iteratable)` returns **a new iterator** object every time when it's called.  Its cursor points to the begining of the list. 
- When **iterator** is the iteratable, `next(iteratable)` returns **the same iterator object** every time when it's called. This iterator's cursor is already travelled to the end of itself after the first loop, and it raises `StopIteration` directly when the second loop starts.


# Conclusion
- An iterable is a class that has `__iter__`  implemented to return an iterator.
- An iterator is a class that has both `__iter__` and `__next__` implemented.
- The for loop calls the `__iter__` method at the background before looping.

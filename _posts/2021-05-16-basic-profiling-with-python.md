---
layout: post
title: "Speeding Up Your Python Code!"
date: 2021-05-16 12:00:00 +0100
categories: [python, development]
---

I'm going to give a demonstration of some basic techniques and principles for optimising your Python code (i.e. making it run faster), with the example of calculating prime numbers under 10000. Let's begin with a straightforward way to calculate primes:

As a reminder, a prime number is a natural number that is greater than 1 and is not the product of two smaller natural numbers.

In order to determine whether a number is the product of two smaller natural numbers, we can use the _modulo_ operator `%`, which given a % b, outputs the remainder of a/b. Here's an example:


```python
print(3 % 2)
print (5 % 5)
print (10 % 4)
```

    1
    0
    2


Now to generate our list of primes, I propose the following algorithm:

- `INITIALIZATION`:
    - Container of prime numbers found thus far. Insert the first prime, 2.
    - Set flag `isPrime` to `True`
- `ITERATION`: FOR all integers `i` greater than 2 and less than 10000:
    - `ITERATION`: FOR each prime number already found `p`
        - `CONDITION`: IF i%p=0:
            - Set flag `isPrime` to `False`
    - `END` ITERATION:
        - `CONDITION`: IF `isPrime` is `True`
            - Append `p` to list of prime numbers
            - Reset flat to initialization
- `END` ITERATION

In words, I check whether each number is divisible with zero remainder by all the primes less than itself. If it is, then I add it to the list of primes.

One might note that there is a lot of redundancy in this algorithm; we will get to that.


```python
isPrime = True
primes = [2] # An empty list

for i in range(3, 10001): # Start from 1, finish at 10000
    # Check if divisible with no remainder from existing arguments
    for p in primes:
        # Here we need some kind of conditional logic. I will use 'if' again.
        if i % p == 0: # If zero remainder, then multiple and not prime
            isPrime = False
        # After all primes checked, if still isPrime=True, then we can append
    if isPrime == True:
        primes.append(i)
    isPrime = True # Resetting the 'trigger'
```

Now here's the first trick for optimising code: the `%%timeit` magic available in iPython. This returns the average of how long it takes to run a code block, trying several runs:


```python
%%timeit

isPrime = True
primes = [2] # An empty list

for i in range(3, 10001): # Start from 1, finish at 10000
    # Check if divisible with no remainder from existing arguments
    for p in primes:
        # Here we need some kind of conditional logic. I will use 'if' again.
        if i % p == 0: # If zero remainder, then multiple and not prime
            isPrime = False
        # After all primes checked, if still isPrime=True, then we can append
    if isPrime == True:
        primes.append(i)
    isPrime = True # Resetting the 'trigger'
```

    354 ms ± 57.8 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


The output of `%%timeit` tells us that the above way of finding all prime numbers less than 10000 takes 281 milliseconds on average, with a standard deviation of 2.23 milliseconds. It also tells us that it ran the code 7 times. (Note that I have am running this on a relatively normal laptop with an i7-8550u (1.8GHz) processor. In fact, the single core speed is relatively slow on this laptop, which is a disadvantage for Python.)

## Step 1: Reducing Redundant Calculations

The code we have here is a _nested for-loop_. As you might now, a for-loop is a structure that runs the same operation for each element of an array. A nested for-loop is a for-loop inside another for-loop. So for each element of the outer array, we run another loop over some other array.

In this example, the outer array is all numbers between 3 and 10000, and the inner array is a growing list of prime numbers. Note that this also means that while the time each operation within the inner loop takes should remain more or less constant, the time taken for each iteration of the outer loop will grow because the length of the inner array increases each time a new prime is found.

In much simpler terms: when there is only one prime, each inner loop (checking i against all prime numbers found thus far) is a single loop. As the number of primes grows, the length of time taken to check each of the primes found thus far accordingly grows. So earlier loops take less time than later ones.

We can reduce the total time taken by decreasing the total number of iterations. An obvious first step is to reduce the outer array by excluding all even numbers; we know that they aren't prime, so no need to check.


```python
%%timeit

isPrime = True
primes = [2]

for i in range(3, 10001, 2): # Third argument is step
    for p in primes:
        if i % p == 0:
            isPrime = False
    if isPrime == True:
        primes.append(i)
    isPrime = True
```

    192 ms ± 37.3 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


That nearly halved the time it took to run, which makes sense; we roughly halved the number of total loops it needed to make.

But the inner loop can be sped up as well. Note that once we find that i is divisible by some p, then it is not necessary to check subsequent values of p. We can end the inner loop early using a `break` statement.


```python
%%timeit

isPrime = True
primes = [2]

for i in range(3, 10001, 2):
    for p in primes:
        if i % p == 0:
            isPrime = False
            break
    if isPrime == True:
        primes.append(i)
    isPrime = True
```

    41.7 ms ± 2.77 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)


We've cut it down again by a factor of 5! (Not five factorial.)

Finally, we can cut down the number of calculations even further by excluding all primes larger than the square root of i. I add another conditional and break:


```python
%%timeit

isPrime = True
primes = [2]

for i in range(3, 10001, 2):
    for p in primes:
        if i % p == 0:
            isPrime = False
            break
        if p > (i**0.5): # Exclude all p > sqrt(i)
            break
    if isPrime == True:
        primes.append(i)
    isPrime = True
```

    6.38 ms ± 338 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)


Without introducing any new libraries or tools, we've sped up our code to be roughly 40 times faster. Moreover, these gains will increase as we try to increase the total number of primes we are trying to find.

## Step 2: Identifying Expensive Calculations

We've eliminated a big chunk of the running time by just removing redundant calculations. The next step is identifying what calculations are taking a lot of time.

Here a knowledge of different libraries and tools is useful. Python provides endless options for numerical arrays, and many boast of their speed and performance.

I will compare four different options for storing integers:

- Lists
- Deques
- `numpy` arrays
- `pandas` Series

Lists are the basic Python mutable array. Deques are another base Python array (from the `collections` library) designed to be faster for appends and other operations on the ends of the array. `numpy` arrays are a standard container for numerical arrays used for numerical computing, and `pandas` is our old-time favourite 1-dimensional data container.

Let's compare the time it takes. (I'll put it into a function to make it a bit more concise).


```python
from collections import deque
import numpy as np
import pandas as pd

def calculate_primes(lower=3, upper=10001, container='list'):
    if container=='list':
        primes = list()
        primes.append(2)
    elif container=='deque':
        primes = deque()
        primes.append(2)
    elif container=='numpy':
        primes = np.array([2])
    elif container=='pandas':
        primes = pd.Series(2)

    isPrime = True
    for i in range(lower, upper, 2):
        for p in primes:
            if i % p == 0:
                isPrime = False
                break
            if p > (i**0.5): # Exclude all p > sqrt(i)
                break
        if isPrime == True:
            if container=='numpy':
                primes = np.append(primes, i)
            elif container=='pandas':
                primes = primes.append(pd.Series(i))
            else:
                primes.append(i)
        isPrime = True
    #return primes
```


```python
%%timeit
calculate_primes(container='list')
```

    6 ms ± 379 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)



```python
%%timeit
calculate_primes(container='deque')
```

	5.9 ms ± 109 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)



```python
%%timeit
calculate_primes(container='numpy')
```

    62.5 ms ± 367 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)



```python
%%timeit
calculate_primes(container='pandas')
```

    376 ms ± 2.93 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


What's going on? `numpy` and `pandas` were much slower than deques or lists!

We can use the `%prun` magic to break this down. This runs a _profiler_, which tracks all the individual function calls.


```python
%prun calculate_primes(container='list')
```

     


             1233 function calls in 0.010 seconds
    
       Ordered by: internal time
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.010    0.010    0.010    0.010 <ipython-input-7-51b6ef22377d>:5(calculate_primes)
         1229    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
            1    0.000    0.000    0.010    0.010 {built-in method builtins.exec}
            1    0.000    0.000    0.010    0.010 <string>:1(<module>)
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}



```python
%prun calculate_primes(container='deque')
```

     


             1233 function calls in 0.007 seconds
    
       Ordered by: internal time
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.007    0.007    0.007    0.007 <ipython-input-7-51b6ef22377d>:5(calculate_primes)
         1229    0.000    0.000    0.000    0.000 {method 'append' of 'collections.deque' objects}
            1    0.000    0.000    0.007    0.007 {built-in method builtins.exec}
            1    0.000    0.000    0.007    0.007 <string>:1(<module>)
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}


We can see that deque and list are pretty similar in terms of the number of function calls. This makes sense, as both are base python objects (and do not rely on underlying objects).


```python
%prun calculate_primes(container='numpy')
```

     


             20881 function calls (18425 primitive calls) in 0.084 seconds
    
       Ordered by: internal time
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
            1    0.067    0.067    0.084    0.084 <ipython-input-7-51b6ef22377d>:5(calculate_primes)
    3684/1228    0.007    0.000    0.016    0.000 {built-in method numpy.core._multiarray_umath.implement_array_function}
         2457    0.002    0.000    0.002    0.000 {built-in method numpy.array}
         1228    0.002    0.000    0.015    0.000 function_base.py:4690(append)
         1228    0.001    0.000    0.004    0.000 fromnumeric.py:1716(ravel)
         1228    0.001    0.000    0.007    0.000 <__array_function__ internals>:2(concatenate)
         1228    0.001    0.000    0.017    0.000 <__array_function__ internals>:2(append)
         1228    0.001    0.000    0.001    0.000 {method 'ravel' of 'numpy.ndarray' objects}
         2456    0.001    0.000    0.003    0.000 _asarray.py:110(asanyarray)
         1228    0.001    0.000    0.005    0.000 <__array_function__ internals>:2(ravel)
         1228    0.000    0.000    0.000    0.000 {built-in method builtins.isinstance}
         1228    0.000    0.000    0.000    0.000 function_base.py:4686(_append_dispatcher)
         1228    0.000    0.000    0.000    0.000 multiarray.py:143(concatenate)
         1228    0.000    0.000    0.000    0.000 fromnumeric.py:1712(_ravel_dispatcher)
            1    0.000    0.000    0.084    0.084 {built-in method builtins.exec}
            1    0.000    0.000    0.084    0.084 <string>:1(<module>)
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}


Whereas the previous examples only ran 1233 calls, `numpy`  requires 20881 calls! It looks like resizing a `numpy` array is quite an expensive thing to do (which we should have already known).


```python
%prun calculate_primes(container='pandas')
```

     


             1125028 function calls (1118884 primitive calls) in 0.679 seconds
    
       Ordered by: internal time
    
       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
       219923    0.054    0.000    0.085    0.000 {built-in method builtins.isinstance}
         2457    0.023    0.000    0.389    0.000 series.py:209(__init__)
        79831    0.020    0.000    0.030    0.000 generic.py:30(_check)
            1    0.020    0.020    0.679    0.679 <ipython-input-7-51b6ef22377d>:5(calculate_primes)
         4913    0.019    0.000    0.036    0.000 {built-in method numpy.core._multiarray_umath.implement_array_function}
         1228    0.018    0.000    0.180    0.000 concat.py:306(__init__)
         1229    0.017    0.000    0.027    0.000 {pandas._libs.lib.maybe_convert_objects}
       119130    0.016    0.000    0.016    0.000 {built-in method builtins.getattr}
         2457    0.015    0.000    0.180    0.000 construction.py:423(sanitize_array)
        17192    0.013    0.000    0.036    0.000 common.py:1470(is_extension_array_dtype)
         1227    0.012    0.000    0.067    0.000 base.py:250(__new__)
        17192    0.012    0.000    0.017    0.000 base.py:411(find)
         2455    0.011    0.000    0.068    0.000 concat.py:101(concat_compat)
        19654    0.011    0.000    0.011    0.000 {built-in method builtins.hasattr}
        13509    0.010    0.000    0.021    0.000 common.py:1610(_is_dtype_type)
         2457    0.009    0.000    0.063    0.000 blocks.py:2662(get_block_type)
         4999    0.009    0.000    0.024    0.000 base.py:796(__iter__)
        19909    0.009    0.000    0.014    0.000 managers.py:1626(internal_values)
         4913    0.008    0.000    0.026    0.000 generic.py:5464(__setattr__)
    42995/36851    0.008    0.000    0.011    0.000 {built-in method builtins.len}
         2455    0.008    0.000    0.013    0.000 concat.py:24(_get_dtype_kinds)
        13509    0.008    0.000    0.036    0.000 base.py:254(is_dtype)
         1228    0.007    0.000    0.235    0.000 concat.py:469(get_result)
         2457    0.007    0.000    0.009    0.000 generic.py:187(__init__)
         1228    0.007    0.000    0.018    0.000 numeric.py:2392(array_equal)
        56495    0.007    0.000    0.007    0.000 {built-in method builtins.issubclass}
         1228    0.007    0.000    0.007    0.000 {method 'reduce' of 'numpy.ufunc' objects}
         2458    0.007    0.000    0.007    0.000 {built-in method numpy.empty}
         1228    0.006    0.000    0.018    0.000 cast.py:1257(maybe_castable)
         8598    0.006    0.000    0.006    0.000 {built-in method numpy.array}
        19909    0.006    0.000    0.021    0.000 series.py:556(_values)
         2457    0.006    0.000    0.013    0.000 blocks.py:127(__init__)
         7370    0.006    0.000    0.017    0.000 base.py:5953(maybe_extract_name)
         7371    0.006    0.000    0.019    0.000 common.py:201(is_object_dtype)
         2457    0.005    0.000    0.083    0.000 blocks.py:2711(make_block)
         1229    0.005    0.000    0.023    0.000 cast.py:1379(maybe_cast_to_datetime)
        11054    0.005    0.000    0.015    0.000 <frozen importlib._bootstrap>:1009(_handle_fromlist)
         6146    0.005    0.000    0.009    0.000 {pandas._libs.lib.is_list_like}
         2457    0.005    0.000    0.093    0.000 managers.py:1577(from_array)
         1229    0.005    0.000    0.020    0.000 range.py:85(__new__)
         1228    0.004    0.000    0.015    0.000 numeric.py:47(__new__)
         4913    0.004    0.000    0.007    0.000 series.py:459(name)
         3684    0.004    0.000    0.014    0.000 dtypes.py:1132(is_dtype)
         1228    0.004    0.000    0.427    0.000 series.py:2585(append)
         2457    0.004    0.000    0.011    0.000 series.py:509(name)
         1229    0.004    0.000    0.009    0.000 cast.py:1617(construct_1d_object_array_from_listlike)
         1228    0.004    0.000    0.063    0.000 construction.py:554(_try_cast)
         6138    0.004    0.000    0.022    0.000 {built-in method builtins.any}
         1228    0.004    0.000    0.122    0.000 base.py:4330(append)
         1228    0.004    0.000    0.012    0.000 _dtype.py:321(_name_get)
         8603    0.004    0.000    0.004    0.000 {built-in method _abc._abc_instancecheck}
         1227    0.004    0.000    0.115    0.000 base.py:4358(_concat)
         9828    0.004    0.000    0.005    0.000 inference.py:322(is_hashable)
         1229    0.003    0.000    0.003    0.000 {built-in method numpy.arange}
         2457    0.003    0.000    0.008    0.000 common.py:231(is_sparse)
         1228    0.003    0.000    0.418    0.000 concat.py:82(concat)
        19909    0.003    0.000    0.003    0.000 managers.py:1588(_block)
         6145    0.003    0.000    0.010    0.000 {built-in method builtins.all}
         1229    0.003    0.000    0.025    0.000 base.py:5947(default_index)
         3684    0.003    0.000    0.013    0.000 dtypes.py:923(is_dtype)
         4911    0.003    0.000    0.015    0.000 common.py:537(is_categorical_dtype)
         1229    0.003    0.000    0.040    0.000 cast.py:109(maybe_convert_platform)
         2457    0.003    0.000    0.003    0.000 blocks.py:269(mgr_locs)
         2457    0.003    0.000    0.003    0.000 generic.py:5446(__getattr__)
         4911    0.003    0.000    0.017    0.000 common.py:499(is_interval_dtype)
         2457    0.003    0.000    0.003    0.000 managers.py:1545(__init__)
         4911    0.003    0.000    0.016    0.000 common.py:463(is_period_dtype)
         2456    0.003    0.000    0.018    0.000 <__array_function__ internals>:2(concatenate)
         3683    0.003    0.000    0.004    0.000 common.py:195(<lambda>)
         7365    0.003    0.000    0.013    0.000 concat.py:139(<genexpr>)
         1229    0.003    0.000    0.007    0.000 range.py:152(_data)
         1229    0.003    0.000    0.008    0.000 range.py:133(_simple_new)
         2456    0.003    0.000    0.008    0.000 generic.py:5519(_protect_consolidate)
        19909    0.002    0.000    0.002    0.000 blocks.py:233(internal_values)
         1229    0.002    0.000    0.010    0.000 numeric.py:288(full)
         9826    0.002    0.000    0.004    0.000 common.py:187(<lambda>)
         2457    0.002    0.000    0.004    0.000 common.py:573(is_string_dtype)
         1228    0.002    0.000    0.027    0.000 cast.py:1677(maybe_cast_to_integer_array)
         8603    0.002    0.000    0.006    0.000 abc.py:137(__instancecheck__)
         2457    0.002    0.000    0.003    0.000 series.py:410(_set_axis)
         9826    0.002    0.000    0.002    0.000 common.py:185(classes)
         1229    0.002    0.000    0.004    0.000 base.py:425(_simple_new)
         1228    0.002    0.000    0.008    0.000 generic.py:5408(__finalize__)
         4910    0.002    0.000    0.002    0.000 concat.py:119(is_nonempty)
         1228    0.002    0.000    0.002    0.000 {method 'astype' of 'numpy.ndarray' objects}
         2458    0.002    0.000    0.002    0.000 base.py:563(_reset_identity)
         2457    0.002    0.000    0.002    0.000 flags.py:47(__init__)
         1228    0.002    0.000    0.131    0.000 concat.py:535(_get_new_axes)
         1228    0.002    0.000    0.127    0.000 concat.py:552(_get_concat_axis)
         6141    0.002    0.000    0.006    0.000 _asarray.py:23(asarray)
         2454    0.002    0.000    0.004    0.000 common.py:429(is_timedelta64_dtype)
         1228    0.002    0.000    0.002    0.000 common.py:64(consensus_name_attr)
         2458    0.002    0.000    0.007    0.000 common.py:1743(validate_all_hashable)
         1228    0.002    0.000    0.007    0.000 concat.py:391(<listcomp>)
         1229    0.002    0.000    0.015    0.000 cast.py:1273(maybe_infer_to_datetimelike)
         2456    0.002    0.000    0.006    0.000 common.py:703(is_integer_dtype)
         1229    0.002    0.000    0.004    0.000 inference.py:263(is_dict_like)
         2456    0.002    0.000    0.003    0.000 numerictypes.py:285(issubclass_)
         3684    0.002    0.000    0.003    0.000 generic.py:5440(<genexpr>)
         1228    0.002    0.000    0.005    0.000 numerictypes.py:359(issubdtype)
         9828    0.002    0.000    0.002    0.000 {built-in method builtins.hash}
         2456    0.002    0.000    0.005    0.000 generic.py:5535(f)
         2457    0.002    0.000    0.007    0.000 construction.py:612(is_empty_data)
         2458    0.002    0.000    0.002    0.000 common.py:157(ensure_python_int)
         2457    0.002    0.000    0.003    0.000 common.py:1769(pandas_dtype)
         2457    0.002    0.000    0.008    0.000 base.py:4070(_values)
         2455    0.002    0.000    0.004    0.000 concat.py:130(<listcomp>)
         4916    0.002    0.000    0.003    0.000 common.py:1762(<genexpr>)
         2457    0.002    0.000    0.007    0.000 construction.py:354(extract_array)
         2456    0.002    0.000    0.009    0.000 generic.py:5531(_consolidate_inplace)
         1228    0.002    0.000    0.006    0.000 _dtype.py:307(_name_includes_bit_suffix)
         2456    0.002    0.000    0.005    0.000 base.py:386(shape)
         4910    0.001    0.000    0.002    0.000 concat.py:135(<genexpr>)
         1228    0.001    0.000    0.123    0.000 concat.py:609(_concat_indexes)
         2457    0.001    0.000    0.008    0.000 common.py:388(is_datetime64tz_dtype)
         3684    0.001    0.000    0.002    0.000 base.py:600(__len__)
         1228    0.001    0.000    0.003    0.000 series.py:397(_constructor_expanddim)
         1228    0.001    0.000    0.003    0.000 common.py:1307(is_float_dtype)
         1229    0.001    0.000    0.002    0.000 common.py:170(all_none)
         1228    0.001    0.000    0.001    0.000 flags.py:83(allows_duplicate_labels)
         2455    0.001    0.000    0.001    0.000 concat.py:138(<setcomp>)
         1228    0.001    0.000    0.128    0.000 concat.py:538(<listcomp>)
         1228    0.001    0.000    0.002    0.000 base.py:4353(<setcomp>)
         2454    0.001    0.000    0.002    0.000 common.py:912(is_datetime64_any_dtype)
         3684    0.001    0.000    0.002    0.000 common.py:1575(get_dtype)
         1227    0.001    0.000    0.004    0.000 common.py:757(is_signed_integer_dtype)
         1228    0.001    0.000    0.001    0.000 {method 'format' of 'str' objects}
         1229    0.001    0.000    0.004    0.000 base.py:1239(name)
         2457    0.001    0.000    0.001    0.000 blocks.py:161(_check_ndim)
         1228    0.001    0.000    0.020    0.000 <__array_function__ internals>:2(array_equal)
         1229    0.001    0.000    0.003    0.000 <__array_function__ internals>:2(copyto)
         1228    0.001    0.000    0.004    0.000 concat.py:482(<listcomp>)
         7366    0.001    0.000    0.001    0.000 {method 'add' of 'set' objects}
         2458    0.001    0.000    0.001    0.000 {built-in method __new__ of type object at 0x559019893240}
         1228    0.001    0.000    0.002    0.000 common.py:615(is_dtype_equal)
         1228    0.001    0.000    0.001    0.000 concat.py:558(<listcomp>)
         1229    0.001    0.000    0.002    0.000 common.py:254(maybe_iterable_to_list)
         2455    0.001    0.000    0.001    0.000 {method 'startswith' of 'str' objects}
         1228    0.001    0.000    0.002    0.000 generic.py:339(_validate_dtype)
         2458    0.001    0.000    0.001    0.000 inference.py:289(<genexpr>)
         2460    0.001    0.000    0.001    0.000 range.py:747(__len__)
         2457    0.001    0.000    0.002    0.000 common.py:1551(_is_dtype)
         1228    0.001    0.000    0.002    0.000 cast.py:1642(construct_1d_ndarray_preserving_na)
         2456    0.001    0.000    0.001    0.000 managers.py:975(consolidate)
         1227    0.001    0.000    0.009    0.000 base.py:4362(<listcomp>)
         1228    0.001    0.000    0.008    0.000 {method 'all' of 'numpy.ndarray' objects}
         1228    0.001    0.000    0.001    0.000 concat.py:529(_get_result_dim)
         3684    0.001    0.000    0.001    0.000 common.py:160(<genexpr>)
         2456    0.001    0.000    0.002    0.000 series.py:2662(<genexpr>)
         3683    0.001    0.000    0.001    0.000 common.py:190(classes_and_not_datetimelike)
         3685    0.001    0.000    0.001    0.000 base.py:1232(name)
         2456    0.001    0.000    0.001    0.000 {built-in method builtins.sum}
         3684    0.001    0.000    0.001    0.000 generic.py:247(flags)
         1228    0.001    0.000    0.001    0.000 common.py:602(condition)
         1228    0.001    0.000    0.007    0.000 _methods.py:59(_all)
         4912    0.001    0.000    0.001    0.000 base.py:397(ndim)
         2458    0.001    0.000    0.001    0.000 common.py:174(<genexpr>)
         1228    0.001    0.000    0.001    0.000 common.py:156(not_none)
         1228    0.001    0.000    0.001    0.000 generic.py:455(_get_axis_number)
         1228    0.001    0.000    0.001    0.000 _dtype.py:24(_kind_name)
         2456    0.000    0.000    0.000    0.000 flags.py:51(allows_duplicate_labels)
         1228    0.000    0.000    0.001    0.000 base.py:5836(ensure_index)
         2457    0.000    0.000    0.000    0.000 numeric.py:78(_validate_dtype)
         2456    0.000    0.000    0.000    0.000 multiarray.py:143(concatenate)
         2457    0.000    0.000    0.000    0.000 blocks.py:265(mgr_locs)
         2458    0.000    0.000    0.000    0.000 {pandas._libs.lib.is_scalar}
         2457    0.000    0.000    0.000    0.000 numeric.py:164(_is_all_dates)
         1229    0.000    0.000    0.000    0.000 range.py:207(start)
         1228    0.000    0.000    0.000    0.000 concat.py:602(_maybe_check_integrity)
         2458    0.000    0.000    0.000    0.000 typing.py:1474(new_type)
         2457    0.000    0.000    0.000    0.000 blocks.py:147(_maybe_coerce_values)
         2457    0.000    0.000    0.000    0.000 typing.py:898(cast)
         2456    0.000    0.000    0.000    0.000 managers.py:1634(is_consolidated)
         1229    0.000    0.000    0.000    0.000 multiarray.py:1054(copyto)
         1229    0.000    0.000    0.000    0.000 range.py:230(stop)
         1228    0.000    0.000    0.000    0.000 numeric.py:2388(_array_equal_dispatcher)
         1228    0.000    0.000    0.000    0.000 series.py:393(_constructor)
         1229    0.000    0.000    0.000    0.000 range.py:253(step)
            1    0.000    0.000    0.679    0.679 {built-in method builtins.exec}
            1    0.000    0.000    0.000    0.000 range.py:694(_concat)
            1    0.000    0.000    0.000    0.000 base.py:510(_shallow_copy)
            1    0.000    0.000    0.000    0.000 base.py:1284(_set_names)
            1    0.000    0.000    0.000    0.000 base.py:1313(set_names)
            1    0.000    0.000    0.679    0.679 <string>:1(<module>)
            3    0.000    0.000    0.000    0.000 range.py:703(<genexpr>)
            1    0.000    0.000    0.000    0.000 numeric.py:107(_shallow_copy)
            1    0.000    0.000    0.000    0.000 range.py:722(<listcomp>)
            1    0.000    0.000    0.000    0.000 base.py:1392(rename)
            1    0.000    0.000    0.000    0.000 range.py:709(<listcomp>)
            1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}


This is a little bit ridiculous. It's clear why it takes fifty times as long to use pandas as base Python in this instance; there are over a million function calls

## Takeaway

The takeaway is not that you should not use `numpy` or `pandas`! Quite the contrary, both of these libraries provide data containers with an enormous number of convenient built-in functionality. Moreover, neither of these containers are optimised with appending in mind; they are very fast for vectorised matrix operations. If we had a different task, they would likely outperform nested lists or deques.

Deques are probably the best base Python object for this kind of task (there may be others I don't know about). But most of our savings came from writing simply more efficient code.

Obviously, writing efficient code takes time and effort. You need to think through all of the little puzzles of your task, and this is a distraction when you are simply trying to prototype some task. At the end of the day, as a relatively new coder, you will spend far more time writing your code than waiting for it to finish executing. This is part of the beauty of Python! It's very easy to prototype code.

My advice is to keep in mind whether steps you are taking are redundant, and to determine whether avoiding such redundancy is simple. If it will only take 5 minutes to implement some logic to avoid millions of pointless operations, then do it! Otherwise consider carefully whether learning `C++` is a good use of your time.

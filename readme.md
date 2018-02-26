# Lazy Code
# Cute and Efficient Is Definitely Possible 

* Bringing the missing elements to C++.
* Compossible, lazily evaluated algorithms for processing data such as map, filter, fold, enumerated ranges and more.
* Easy handling of input and output streams, parsing integers, reading line by line, etc.
* Compact syntax showing clear flow of data.
* Zero cost abstractions leveraging C++ templates.
* _Optionally_ enabled  macros to make the syntax even more cute.


## Example 1:

Read in lines of text into a vector, one line per element, until EOF is reached.  Sort the vector and then re print the lines.

```c++
    auto lines = readLines(cin) | vector<string>();
    sort(lines.begin(), lines.end());
    lines | write(cout, "\n");
    lines | write(cout, "\n");
```

Yup, that's it.  And it gets better...

## Example 2:

Read in up to 10 integers from standard input.  Skip odd integers.  Square the remaining numbers and sum their values.  Use minimal memory, i.e. do not store the numbers in a vector, after all, it could easily be more than 10 numbers.

```c++
int total = read<int>(cin) | limit(10) |
            filter([](int i) { return i % 2 == 0; }) |
            map([](int i) { return i * i; }) | sum();
```

__Wow__, that's compact.  Maybe too compact? If you are concerned, you can split that line up into multiple expressions.  Take a look:

```c++
    auto numbers = read<int>(cin) | limit(10);
    auto evenFilter = numbers | filter([](int i) { return i % 2 == 0; });
    auto squares = evenFilter | map([](int i) { return i * i; });
    int total = squares | sum();
    cout << total << endl;
```

* Even though this expression is split over multiple lines, it is not any less efficient.  * The numbers are not stored in any intermediate container.
* The final value (the sum of the squares of only even numbers) is calculated in one go, in a single loop from `standard in` to the `total` variable.
* Writing the equivalent loop is more cumbersome and is more likely to contain errors.  For example, this has a bug.  Can you spot it?

```c++
    int total = 0;
    for (int i = 0; i < 10; i++) {
        int value;
        cin >> value;
        if (value % 2 == 0) {
            total += value * value;
        }
    }
```

__The bug?__  What if the user enters less than 10 numbers.  You'll be reading EOF symbols into your total.


## Even more cute:

Those long lambdas, what can we do?  You could use a macro *don't panic, macros are optional, I only offer a single one for convenience*.

```c++
    int total = read<int>(cin) | limit(10) | filter(lambda(i, i % 2 == 0)) |
                map(lambda(i, i * i)) | sum();
    cout << total << endl;
```


Now that we've sorted that out, from now on, examples are shown with the lambda macro enabled.  

# The docs:

Here we cover the API.

* Ranges, evaluators and piping:
* Number ranges
* Enumerate
* Mapping
* Filtering
* folding
* Reading from streams
* Writing  to streams


## Behind the scenes; Ranges, evaluators and piping:

This gives some behind the scenes information. if you just want to get writing quickly, skip to the next section.

Under all those cute expressions you'll be writing with *lazyCode* are two categories of objects; ranges and evaluators.  Ranges are objects that represent some kind of iterable.  They are used to generate a stream of values.  Most of the objects like map, filter, fold, zip, etc return ranges.  Ranges can be composed together to produce new ranges, usually with the pipe `|` symbol.  For example, given a filter range `a` and a map range `b`, you can produce a range that filters then maps (in one go) with `a | b`.  The same notation works even if `a`` is an STL  container (e.g. std::vector).  The container is wrapped in a range designed for iterating over containers.

By default, ranges are lazily evaluated.  This means that unless explicitly invoked, ranges simply sit where they have been created, incurring no runtime costs.  There are three main ways to pull values out of ranges:

1.  Use a range based for loop, `for (auto i: rangeObj) { ... }`.  Since ranges support `begin()` and `end()` they can be used anywhere input iterators may be used.
1.  Pipe the range into an evaluator (see below).
1.  Use the range interface (least recommended and not guarantied to be stable).

Evaluators turn ranges into results.  When piping a range into an evaluator, LazyCode pulls all the values from the range and pushes them to the evaluator.  Evaluators may then return a result based on the pushed values.  For example, `count()`  returns the number of values pushed, `write()` writes the pushed values to a stream, etc.  Evaluators are used by composing them with ranges in the same notation `range | evaluator`.

Unless stated otherwise, whenever an object  `a`  is composed with another `b`, if `a` is a variable (lvalue/lvalue reference), `b` will only hold a reference to `a`.  However, if `a` is a temporary object (rvalue), which usually happens when you create a range inline, then `b` will own `a`.  That is, if `a` is a temporary, `a` will be moved into `b` such that `a` remains constructed until `b` is destructed.

## Jumping write in;  let's begin with Number ranges.

The most simple but often useful range, iterating over numbers.


```c++
    // iterate 0..5 (5 exclusive)
    for (int x : range(5)) {
    }
    // iterate 10..20
    for (int x : range(10, 20)) {
    }
    // iterate 0..10 in steps of 2
    for (int x : range(0, 10, 2)) {
    }
    // iterate 1.0 to 2.0 in increments of 0.1
    for (double x : range(1.0, 2.0, 0.1)) {
    }
    //iterate backwards (negative ranges not supported yet
```

## Piping:

All the above can be piped directly into containers.

```c++
    // pipe into new vector
    auto vec1 = range(5) | vector<int>();
    // pipe into existing vector
    vector<int> vec2;
    range(5) | vec2;
```


tbc 
Advanced R - Exercises and Notes
================
Josh Livingston \|
June 27, 2022

<p>This repository stores all of my notes and exercise work-throughs from <a href="https://adv-r.hadley.nz/">Advanced R</a>. This uses the 2nd edition of Hadley Wickham's book.</p>
<p>Source code is organized at the chapter-section level. In each section, notes appear first, followed by the exercises. Exercise question text are written in <em>italics</em>.</p>
<p>This README was rendered using <a href="https::quarto.org">quarto</a>.</p>


# 2: Names and values

## 2.2: Binding basics

### Notes

-   A value does not have a name; a name has a value.
-   That is, a name gets bound to a value as a reference to that value.
-   Assigning a second name to that object does not change the object or
    copy it. It simply assigns a new name as reference to that object.
-   Names have to be pretty. See `?make.names` for rules governing
    syntactically valid names.

### Exercises

<em>1. Explain the relationship between `a`, `b`, `c`, and `d` in the
following code:\*

``` r
a <- 1:10
b <- a
c <- b
d <- 1:10
```

`a`, `b`, and `c` all occupy the same space in memory. Thus, they are
separate references, each of which have been assigned to the same
object.

``` r
library(lobstr)
obj_addr(a) == obj_addr(b) & obj_addr(b) == obj_addr(c)
```

    [1] TRUE

``` r
print(obj_addr(a))
```

    [1] "0x7fa3ba10d968"

<br>

However, `d` does not occupy the same memory space. Thus, `d` is an is a
separate object from that of `a`, `b`, and `c`, with a unique reference.
Both of these objects are the vector of integers 1 through 10.

``` r
obj_addr(d) == obj_addr(a)
```

    [1] FALSE

``` r
print(obj_addr(d))
```

    [1] "0x7fa3b9d4edd8"

<br>

*2. The following code accesses the mean function in multiple ways. Do
they all point to the same underlying function object?*

All accessors to the `mean()` function point to the same object in
memory.

``` r
objs <- list(mean, base::mean, evalq(mean), match.fun("mean"))
obj_addrs(objs)
```

    [1] "0x7fa3b9a17860" "0x7fa3b9a17860" "0x7fa3b9a17860" "0x7fa3b9a17860"

<br>

*3. By default, base R data import functions, like `read.csv()`, will
automatically convert non-syntactc names to syntactic ones. Why might
this be problematic? What option allows you to supporess this behavior?*

Column names often represent data, so renaming with `make.names` changes
underlying data. You can suppress this with `check.names = FALSE`.
<br><br>

*4. What rules does `make.names()` use to convert non-syntactic names
into syntactic ones?*

From the docs, *“A syntactically valid name consists of letters, numbers
and the dot or underline characters and starts with a letter or the dot
not followed by a number.”* Letters are defined by locale, but only
ASCII digits are used. Invalid characters are translated to “.”. Missing
is translated to “NA”. And reserved words have a “.” appended to them.
Then, values are de-duplicated using `make.unique()`. <br><br>

*5. I slightly simplified the rules that govern syntactic names. Why is
`.123e1` not a syntactic name?*

Syntactic names may start with a letter, or a dot not followed by a
number. `.123e1` starts with `.1`, so it is not a syntactically valid
name. <br><br>

## 2.3: Copy-on-modify

### Notes

Consider the following two variables

``` r
x <- c(1, 2, 3)
y <- x
```

<br>

Note from previously that `x` and `y` are different references to the
same object.

``` r
obj_addr(x) == obj_addr(y)
```

    [1] TRUE

<br>

This object is located at the following address:

``` r
obj_addr(y)
```

    [1] "0x7fa3aa9791a8"

<br>

Modiyfing the object assigned to `y` results in the creation of a new
object.

``` r
y[[3]] <- 4
obj_addr(y)
```

    [1] "0x7fa3a9108658"

We see that this is different than the original object’s address

``` r
obj_addr(x) == obj_addr(y)
```

    [1] FALSE

This behavior is called ***copy-on-modify***; i.e., R objects are
immutable – any changes results in the creation of a new object in
memory.

#### tracemem()

`base::tracemem()` will track an object’s location in memory.

``` r
x <- c(1, 2, 3)
cat(tracemem(x), "\n")
```

    <0x7fa3d9b75d18> 

In the example below, a second name, `y` was assigned to an object,
which already had an assigned name `x`. So when `x` or `y` is modified,
copy-on-modify takes place. This will trigger `tracemem()` to print a
memory change.

``` r
y <- x
y[[4]] <- 4L
```

    tracemem[0x7fa3d9b75d18 -> 0x7fa3ba3660c8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

`base::untracemem()` is the opposite of `base::tracemem()`

``` r
untracemem(x)
```

#### Lists

Lists do not store values. Instead, they store references to them.

When you modify elements of a list, you can view how this effects
copy-on-modify behavior using `lobstr::ref()`.

Here, we assign a new reference to a list object. We then modify one of
the elements in the list, triggering copy-on-modify. iwth
`lobstr::ref()`, we see that a new object was created for the list
itself and the modified object. The other elements of the list remain
the same object.

In the output below, the first and fifth references are the lists
themselves. The sixth references points to the new object, 4.

``` r
l1 <- list(1, 2, 3)
l2 <- l1
l1[[3]] <- 4
ref(l1, l2)
```

    █ [1:0x7fa3caf20028] <list> 
    ├─[2:0x7fa3cad9d138] <dbl> 
    ├─[3:0x7fa3cad9d100] <dbl> 
    └─[4:0x7fa3cad9cfe8] <dbl> 
     
    █ [5:0x7fa3caec2cf8] <list> 
    ├─[2:0x7fa3cad9d138] 
    ├─[3:0x7fa3cad9d100] 
    └─[6:0x7fa3cad9d0c8] <dbl> 

#### Data frames

Since data frames are a list of columns, and those columns are vectors,
modifying a column only results in that column being copied:

``` r
d1 <- data.frame(a = c(1, 2, 3), b = c(4, 5, 6))
tracemem(d1)
```

    [1] "<0x7fa3aaccd888>"

Here, `tracemem()` shows us that the new column was copied to a new
object in memory.

``` r
d2 <- d1
d2[, 2] <- d2[, 2] * 2
```

    tracemem[0x7fa3aaccd888 -> 0x7fa3ba76cc88]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3ba76cc88 -> 0x7fa3ba76cbc8]: [<-.data.frame [<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

And with `lobstr::ref()`, we confirm that both the data.frame object and
the second column were copied.

``` r
ref(d1, d2)
```

    tracemem[0x7fa3aaccd888 -> 0x7fa3ba5f0248]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3ba76cbc8 -> 0x7fa3ba3ebe88]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

    █ [1:0x7fa3aaccd888] <df[,2]> 
    ├─a = [2:0x7fa3aacd1018] <dbl> 
    └─b = [3:0x7fa3aacd0fc8] <dbl> 
     
    █ [4:0x7fa3ba76cbc8] <df[,2]> 
    ├─a = [2:0x7fa3aacd1018] 
    └─b = [5:0x7fa3aace7d08] <dbl> 

Since data.frames are built column-wise, modifying a row results in
copying every column.

``` r
d3 <- d1
d1[1, ] <- d1[1, ] * 2
```

    tracemem[0x7fa3aaccd888 -> 0x7fa3ba7d9988]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3ba7d9988 -> 0x7fa3ba7d96c8]: [<-.data.frame [<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(d1)
ref(d1, d3)
```

    tracemem[0x7fa3aaccd888 -> 0x7fa3cabf1348]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

    █ [1:0x7fa3ba7d96c8] <df[,2]> 
    ├─a = [2:0x7fa3d9c03b48] <dbl> 
    └─b = [3:0x7fa3d9c03878] <dbl> 
     
    █ [4:0x7fa3aaccd888] <df[,2]> 
    ├─a = [5:0x7fa3aacd1018] <dbl> 
    └─b = [6:0x7fa3aacd0fc8] <dbl> 

#### Character vectors

R uses a global string pool in each session. This means that each
element of a character vector points to a string in the globally unique
pool. The references can be viewed in `lobstr::ref()` by setting
`character` to `TRUE`.

``` r
x <- letters[1:3]
ref(x, character = TRUE)
```

    █ [1:0x7fa3a9465bf8] <chr> 
    ├─[2:0x7fa3c904dae8] <string: "a"> 
    ├─[3:0x7fa3d96214e8] <string: "b"> 
    └─[4:0x7fa3d980dac0] <string: "c"> 

### Exercises

*1. Why is `tracemem(1:10)` not useful?*

`1:10` is a sequence no name assigned to it, therefore will not be
traceable after this initial call.

*2. Explain why `tracemem()` shows two copies when you run this code.
Hint: carefully look at the difference between this code and the code
shown earlier in the section.*

``` r
x <- c(1L, 2L, 3L)
tracemem(x)
```

    [1] "<0x7fa3d9eb2988>"

``` r
x[[3]] <- 4
```

    tracemem[0x7fa3d9eb2988 -> 0x7fa3d9eea148]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3d9eea148 -> 0x7fa3a9308ef8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(x)
```

In this example, the original object `x` is an integer vector. When the
third element of the vector is modifed to 4, a double, the vector is
first modified by being converted to a double vector. Then it is
modified again when the third element is modified. This results in two
copies-on-modify, reflected in the `tracemem()` output.

## 2.4: Object size

### Notes

`lobstr::obj_size()` shows you the size of an object in memory.

``` r
obj_size(iris)
```

    7,200 B

``` r
obj_size(c(1, 2, 3))
```

    80 B

#### Lists

Because lists store refereneces to objects, a repeated list is smaller
than one would imagine. This is because the reference occupies less
space in memeory than the object itself.

``` r
l1 <- list(rep(100, 100000))
obj_size(l1)
```

    800,104 B

``` r
l2 <- list(l1, l1, l1)
obj_size(l2)
```

    800,184 B

A list repeated three times is 80 bytes larger than the original list,
which was of size 800,104 bytes. 80 bytes is the size of a list of three
empty objects:

``` r
obj_size(list(NULL, NULL, NULL))
```

    80 B

#### Character vectors

Similarly, since character vectors are references to the global string
pool, a repeated character vector does not increase size dramatically:

``` r
s <- "ask me about my sentence"
obj_size(s)
```

    136 B

``` r
obj_size(rep(s, 10))
```

    256 B

#### Alternative representation

Starting in R 3.5.0, alternative representation allowed R to represent
certain vectors compactly. This is most commonly observed when using `:`
to create a sequence.

Because of ALTREP, every sequence in R has the same size, regardless of
the sequence length:

``` r
obj_size(1:2)
```

    680 B

``` r
obj_size(1:10)
```

    680 B

``` r
obj_size(1:1e9)
```

    680 B

### Exercises

*1.In the following example, why are object.size(y) and obj_size(y) so
radically different?*

``` r
y <- rep(list(runif(1e4)), 100)

object.size(y)
```

    8005648 bytes

``` r
obj_size(y)
```

    80,896 B

Per the documentation, `base::object.size()` measures the size of each
object, and does not account for shred objects within lists.

*2. Take the following list. Why is its size somewhat misleading?*

``` r
funs <- list(mean, sd, var)
obj_size(funs)
```

    17,608 B

These objects come shipped with base R, so they are always available. It
does not represent additional memory allocated to these objects.

## 2.5: Modify-in-place

### Notes

There are two exceptions to copy-on-modify:

-   Objects with a single binding
-   Environments

#### Objects with a single binding

If an object has only one name assigned to it, R will modify in place.

Before:

``` r
a <- c(1, 2, 3)
ref(a)
```

    [1:0x7fa3aa8b6e38] <dbl> 

After:

``` r
a[[3]] <- 4
ref(a)
```

    [1:0x7fa3aaac43e8] <dbl> 

Note that this optimization does *not* apply when modifyins a vector’s
length. Here, `z` is assigned to a new object upon “adding” a fourth
element to the vector `z`.

Before:

``` r
z <- letters[1:3]
obj_addr(z)
```

    [1] "0x7fa3a9142e68"

<br>

After:

``` r
z[[4]] <- "d"
obj_addr(z)
```

    [1] "0x7fa3a911f258"

There are two complications in R’s behavior that limit execution of the
modify-in-place optimization:

1.  R can only count if an object has 0, 1, or many references. That
    means that if an object has one of two bindings removed, the binding
    count will remain at many and modify-in-place will not apply.
2.  Most functions make a reference of the object to be modified, so
    copy-on-modify would apply. The exception are “primitive” C
    functions, found mostly in the base package.

For example, modifying a data frame results in additional references
being made, whereas modifying a list uses internal C code that does not
create new references.

It’s best to confirm copy behavior using `tracemem()`.

First, on modifying a data frame:

``` r
x <- as.data.frame(matrix(runif(1e3), ncol = 4))
tracemem(x)
```

    [1] "<0x7fa3cac229d8>"

``` r
for (i in seq_along(x)) {
  x[[i]] <- x[[i]] * 5
}
```

    tracemem[0x7fa3cac229d8 -> 0x7fa3aacc7238]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3aacc7238 -> 0x7fa3aaccd068]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3aaccd068 -> 0x7fa3aacccfc8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3aacccfc8 -> 0x7fa3aaccce88]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3aaccce88 -> 0x7fa3aacccd98]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3aacccd98 -> 0x7fa3aacccca8]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3aacccca8 -> 0x7fa3aacccc08]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fa3aacccc08 -> 0x7fa3aacccb18]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(x)
```

Then a list:

``` r
l <- as.list(x)
tracemem(l)
```

    [1] "<0x7fa3a9137f18>"

``` r
for (i in seq_along(l)) {
  l[[i]] <- l[[i]] * 5
}
```

    tracemem[0x7fa3a9137f18 -> 0x7fa3aacb8428]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(l)
```

#### Environments

Environments are always modified-in-place, since existing bindings in
that environment continue to have the same reference.

### Exercises

*1.Explain why the following code doesn’t create a circular list.*

``` r
x <- list()
x[[1]] <- x
```

In the first line of code, an empty list is created, and the name `x`
assigned to it.

The next line of code modifies the length of `x`, so copy-on-modify
takes place to create a new list of length one, with the first element
being `x`, the original empty list.

This is confirmed by observing the matching memory addresses:

Original list:

``` r
x <- list()
ref(x)
```

    █ [1:0x7fa3ba495e30] <list> 

Modified list:

``` r
x[[1]] <- x
ref(x)
```

    █ [1:0x7fa3d94ae6e8] <list> 
    └─█ [2:0x7fa3ba495e30] <list> 

*2. Wrap the two methods for subtracting medians into two functions,
then use the ‘bench’ package to carefully compare their speeds. How does
performance change as the number of columns increase?*

Note: Instead of multiplying columns by 5 to demonstrate the exception
to modify-in-place, the book subtracted medians. I’ll continue to
multiply by 5 below.

With a 250 x 4 data set, the differences in both speed is about 6-8x.

``` r
library(bench)
mult_five_seq <- function(x) {
  for (i in seq_along(x)) {
    x[[i]] <- x[[i]] * 5
  }
}

df <- as.data.frame(matrix(runif(1e4), ncol = 4))
l <- as.list(df)

bm <- mark(df = mult_five_seq(df), l = mult_five_seq(l))
knitr::kable(bm[, 1:5])
```

| expression |     min |  median |   itr/sec | mem_alloc |
|:-----------|--------:|--------:|----------:|----------:|
| df         | 45.17µs | 48.33µs |  19053.75 |    99.9KB |
| l          |  3.67µs |  4.96µs | 178270.23 |    78.3KB |

With a 250 x 400 data set, the differences in both speed and memory
allocation are much more pronounced. Here, 70x faster and a quarter of
the memory used.

``` r
df <- as.data.frame(matrix(runif(1e5), ncol = 400))
l <- as.list(df)

bm <- mark(df = mult_five_seq(df), l = mult_five_seq(l))
knitr::kable(bm[, 1:5])
```

| expression |     min | median |   itr/sec | mem_alloc |
|:-----------|--------:|-------:|----------:|----------:|
| df         |  7.73ms | 8.02ms |  122.4801 |    3.26MB |
| l          | 96.58µs |  103µs | 9193.3565 |  803.17KB |

*3. What happens if you attempt to use `tracemem()` on an environment?*
Modify-in-place always applies to environments, since existing bindings
keep their references.

## 2.6: Unbinding the garbage collector

### Notes

R uses a **garbage collector** to automatically free up unused memory
when it is needed for a new object. You can force a garbage collection
with `gc()`, but there’s never any need to call it.

An example of unused memory are objects that no longer have a reference.

``` r
x <- 1:3
x <- 2:4
rm(x)
```

Two objects were created above as a result of the original object
creation and the copy-on-modify behavior. Then, the name `x` was
removed, but the objects remained. R’s garbage collector will remove
these objects when necessary. You can have the collector print a message
every time it runs with `gcinfo(TRUE)`.

# 3: Vectors

# 4: Subsetting

# 5: Control Flow

# 6: Functions

# 7: Environments

# 8: Conditions

# 9: Functionals

# 10: Function factories

# 11: Function operators

# 12: Base types

# 13: S3

# 14: R6

# 15: S4

# 16: Trade-offs

# 17: Big picture

# 18: Expressions

# 19: Quasiquotation

# 20: Evaluation

# 21: Translating R code

# 22: Debugging

# 23: Measuring performance

# 24: Improving performance

# 25: Rewriting R code in C++

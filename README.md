Advanced R - Exercises and Notes
================
Josh Livingston \|
June 28, 2022

<p>This repository stores all of my notes and exercise work-throughs from <a href="https://adv-r.hadley.nz/">Advanced R</a>. This uses the 2nd edition of Hadley Wickham's book.</p>
<p>Source code is organized at the chapter-section level. In each section, notes appear first, followed by the exercises. Exercise question text are written in <em>italics</em>.</p>
<p>This README was rendered using <a href="https::quarto.org">quarto</a>.</p>


# 2: Names and values

[Book Link](https://adv-r.hadley.nz/names-values.html#names-values)

## 2.2: Binding basics

### Notes

-   A value does not have a name; a name has a value.
-   That is, a name gets bound to a value as a reference to that value.
-   Assigning a second name to that object does not change the object or
    copy it. It simply assigns a new name as reference to that object.
-   Names have to be pretty. See `?make.names` for rules governing
    syntactically valid names.

### Exercises

*1. Explain the relationship between `a`, `b`, `c`, and `d` in the
following code:*

``` r
a <- 1:10
b <- a
c <- b
d <- 1:10
```

<br> `a`, `b`, and `c` all occupy the same space in memory. Thus, they
are separate references, each of which have been assigned to the same
object.

``` r
library(lobstr)
obj_addr(a) == obj_addr(b) & obj_addr(b) == obj_addr(c)
```

    [1] TRUE

``` r
print(obj_addr(a))
```

    [1] "0x7f8f94525fb0"

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

    [1] "0x7f8f92b01780"

<br>

*2. The following code accesses the mean function in multiple ways. Do
they all point to the same underlying function object?*

All accessors to the `mean()` function point to the same object in
memory.

``` r
objs <- list(mean, base::mean, evalq(mean), match.fun("mean"))
obj_addrs(objs)
```

    [1] "0x7f8f72a31660" "0x7f8f72a31660" "0x7f8f72a31660" "0x7f8f72a31660"

<br>

*3. By default, base R data import functions, like `read.csv()`, will
automatically convert non-syntactic names to syntactic ones. Why might
this be problematic? What option allows you to suppress this behavior?*

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

    [1] "0x7f8f6434da18"

<br>

Modifying the object assigned to `y` results in the creation of a new
object.

``` r
y[[3]] <- 4
obj_addr(y)
```

    [1] "0x7f8f94360f38"

<br> We see that this is different than the original object’s address

``` r
obj_addr(x) == obj_addr(y)
```

    [1] FALSE

<br> This behavior is called ***copy-on-modify***; i.e., R objects are
immutable – any changes results in the creation of a new object in
memory.

#### tracemem()

`base::tracemem()` will track an object’s location in memory.

``` r
x <- c(1, 2, 3)
cat(tracemem(x), "\n")
```

    <0x7f8f92874788> 

<br> In the example below, a second name, `y` was assigned to an object,
which already had an assigned name `x`. So when `x` or `y` is modified,
copy-on-modify takes place. This will trigger `tracemem()` to print a
memory change.

``` r
y <- x
y[[4]] <- 4L
```

    tracemem[0x7f8f92874788 -> 0x7f8f92b43268]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

<br> `base::untracemem()` is the opposite of `base::tracemem()`

``` r
untracemem(x)
```

#### Lists

Lists do not store values. Instead, they store references to them.

When you modify elements of a list, you can view how this effects
copy-on-modify behavior using `lobstr::ref()`.

Here, we assign a new reference to a list object. We then modify one of
the elements in the list, triggering copy-on-modify. With
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

    █ [1:0x7f8f73765d28] <list> 
    ├─[2:0x7f8f7377ad18] <dbl> 
    ├─[3:0x7f8f7377ace0] <dbl> 
    └─[4:0x7f8f7377abc8] <dbl> 
     
    █ [5:0x7f8f73602f78] <list> 
    ├─[2:0x7f8f7377ad18] 
    ├─[3:0x7f8f7377ace0] 
    └─[6:0x7f8f7377aca8] <dbl> 

#### Data frames

Since data frames are a list of columns, and those columns are vectors,
modifying a column only results in that column being copied:

``` r
d1 <- data.frame(a = c(1, 2, 3), b = c(4, 5, 6))
tracemem(d1)
```

    [1] "<0x7f8f947d2d88>"

<br> Here, `tracemem()` shows us that the new column was copied to a new
object in memory.

``` r
d2 <- d1
d2[, 2] <- d2[, 2] * 2
```

    tracemem[0x7f8f947d2d88 -> 0x7f8f645a1788]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f645a1788 -> 0x7f8f645a16c8]: [<-.data.frame [<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

<br> And with `lobstr::ref()`, we confirm that both the data.frame
object and the second column were copied.

``` r
ref(d1, d2)
```

    tracemem[0x7f8f947d2d88 -> 0x7f8f92d81248]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f645a16c8 -> 0x7f8f82f2ce48]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

    █ [1:0x7f8f947d2d88] <df[,2]> 
    ├─a = [2:0x7f8f947d5728] <dbl> 
    └─b = [3:0x7f8f947d56d8] <dbl> 
     
    █ [4:0x7f8f645a16c8] <df[,2]> 
    ├─a = [2:0x7f8f947d5728] 
    └─b = [5:0x7f8f645a5618] <dbl> 

<br> Since data.frames are built column-wise, modifying a row results in
copying every column.

``` r
d3 <- d1
d1[1, ] <- d1[1, ] * 2
```

    tracemem[0x7f8f947d2d88 -> 0x7f8f731c63c8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f731c63c8 -> 0x7f8f731c60c8]: [<-.data.frame [<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

<br>

``` r
untracemem(d1)
ref(d1, d3)
```

    tracemem[0x7f8f947d2d88 -> 0x7f8f73625608]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

    █ [1:0x7f8f731c60c8] <df[,2]> 
    ├─a = [2:0x7f8f638e7ec8] <dbl> 
    └─b = [3:0x7f8f638e7e28] <dbl> 
     
    █ [4:0x7f8f947d2d88] <df[,2]> 
    ├─a = [5:0x7f8f947d5728] <dbl> 
    └─b = [6:0x7f8f947d56d8] <dbl> 

#### Character vectors

R uses a global string pool in each session. This means that each
element of a character vector points to a string in the globally unique
pool. The references can be viewed in `lobstr::ref()` by setting
`character` to `TRUE`.

``` r
x <- letters[1:3]
ref(x, character = TRUE)
```

    █ [1:0x7f8f644d1518] <chr> 
    ├─[2:0x7f8f9385cce8] <string: "a"> 
    ├─[3:0x7f8f92e5a0e8] <string: "b"> 
    └─[4:0x7f8f9280acc0] <string: "c"> 

### Exercises

*1. Why is `tracemem(1:10)` not useful?*

`1:10` is a sequence no name assigned to it, therefore will not be
traceable after this initial call. <br><br>

*2. Explain why `tracemem()` shows two copies when you run this code.
Hint: carefully look at the difference between this code and the code
shown earlier in the section.*

``` r
x <- c(1L, 2L, 3L)
tracemem(x)
```

    [1] "<0x7f8f640cce48>"

``` r
x[[3]] <- 4
```

    tracemem[0x7f8f640cce48 -> 0x7f8f640f8448]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f640f8448 -> 0x7f8f644fbf68]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(x)
```

<br> In this example, the original object `x` is an integer vector. When
the third element of the vector is modified to 4, a double, the vector
is first modified by being converted to a double vector. Then it is
modified again when the third element is modified. This results in two
copies-on-modify, reflected in the `tracemem()` output. <br><br>

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

Because lists store references to objects, a repeated list is smaller
than one would imagine. This is because the reference occupies less
space in memory than the object itself.

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

<br> A list repeated three times is 80 bytes larger than the original
list, which was of size 800,104 bytes. 80 bytes is the size of a list of
three empty objects:

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

<br>

Per the documentation, `base::object.size()` measures the size of each
object, and does not account for shred objects within lists. <br><br>

*2. Take the following list. Why is its size somewhat misleading?*

``` r
funs <- list(mean, sd, var)
obj_size(funs)
```

    17,608 B

<br> These objects come shipped with base R, so they are always
available. It does not represent additional memory allocated to these
objects. <br><br>

## 2.5: Modify-in-place

### Notes

There are two exceptions to copy-on-modify:

-   Objects with a single binding
-   Environments

In these exceptions, R executes a modify in place optimization.

#### Objects with a single binding

If an object has only one name assigned to it, R will modify in place.

Before:

``` r
a <- c(1, 2, 3)
ref(a)
```

    [1:0x7f8f642dc6b8] <dbl> 

<br> After:

``` r
a[[3]] <- 4
ref(a)
```

    [1:0x7f8f644b5ce8] <dbl> 

<br> Note that this optimization does *not* apply when modifying a
vector’s length. Here, `z` is assigned to a new object upon “adding” a
fourth element to the vector `z`.

Before:

``` r
z <- letters[1:3]
obj_addr(z)
```

    [1] "0x7f8f644e3e28"

<br>

After:

``` r
z[[4]] <- "d"
obj_addr(z)
```

    [1] "0x7f8f644f7ce8"

<br> There are two complications in R’s behavior that limit execution of
the modify-in-place optimization:

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

    [1] "<0x7f8f947d94f8>"

``` r
for (i in seq_along(x)) {
  x[[i]] <- x[[i]] * 5
}
```

    tracemem[0x7f8f947d94f8 -> 0x7f8f947d5d18]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f947d5d18 -> 0x7f8f947d4fa8]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f947d4fa8 -> 0x7f8f947d4f08]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f947d4f08 -> 0x7f8f947d4dc8]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f947d4dc8 -> 0x7f8f947d4cd8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f947d4cd8 -> 0x7f8f947d4be8]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f947d4be8 -> 0x7f8f947d4b48]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7f8f947d4b48 -> 0x7f8f947d4a58]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(x)
```

<br> Then a list:

``` r
l <- as.list(x)
tracemem(l)
```

    [1] "<0x7f8f65058bc8>"

``` r
for (i in seq_along(l)) {
  l[[i]] <- l[[i]] * 5
}
```

    tracemem[0x7f8f65058bc8 -> 0x7f8f650600d8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(l)
```

#### Environments

Environments are always modified-in-place, since existing bindings in
that environment continue to have the same reference.

### Exercises

*1. Explain why the following code doesn’t create a circular list.*

``` r
x <- list()
x[[1]] <- x
```

<br> In the first line of code, an empty list is created, and the name
`x` assigned to it.

The next line of code modifies the length of `x`, so copy-on-modify
takes place to create a new list of length one, with the first element
being `x`, the original empty list.

This is confirmed by observing the matching memory addresses:

Original list:

``` r
x <- list()
ref(x)
```

    █ [1:0x7f8f82d87cc8] <list> 

<br> Modified list:

``` r
x[[1]] <- x
ref(x)
```

    █ [1:0x7f8fa2babd20] <list> 
    └─█ [2:0x7f8f82d87cc8] <list> 

<br>

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

| expression |     min | median |  itr/sec | mem_alloc |
|:-----------|--------:|-------:|---------:|----------:|
| df         | 45.96µs | 50.4µs |  18344.4 |    99.9KB |
| l          |  3.71µs |    5µs | 183542.9 |    78.3KB |

<br> With a 250 x 400 data set, the differences in both speed and memory
allocation are much more pronounced. Here, 70x faster and a quarter of
the memory used.

``` r
df <- as.data.frame(matrix(runif(1e5), ncol = 400))
l <- as.list(df)

bm <- mark(df = mult_five_seq(df), l = mult_five_seq(l))
knitr::kable(bm[, 1:5])
```

| expression |     min |   median |   itr/sec | mem_alloc |
|:-----------|--------:|---------:|----------:|----------:|
| df         |  7.83ms |   8.01ms |  124.2255 |    3.26MB |
| l          | 94.88µs | 106.54µs | 8402.4141 |  803.17KB |

<br> *3. What happens if you attempt to use `tracemem()` on an
environment?* Modify-in-place always applies to environments, since
existing bindings keep their references. <br><br>

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

<br> Two objects were created above as a result of the original object
creation and the copy-on-modify behavior. Then, the name `x` was
removed, but the objects remained. R’s garbage collector will remove
these objects when necessary. You can have the collector print a message
every time it runs with `gcinfo(TRUE)`. <br><br>

# 3: Vectors

[Book Link](https://adv-r.hadley.nz/vectors-chap.html#vectors-chap)

-   Vectors, an important R data type, have two types: *atomic vectors*
    and *lists* (generic vectors)
-   Vectors also have metadata in the form of *attributes*

## 3.2: Atomic Vectors

### Notes

-   Four primary types of atomic vectors
    -   logical
    -   character
    -   double
    -   integer
        -   Together, double and integer are numeric vectors
-   Two rare types:
    -   complex
    -   raw

#### Scalars

Scalars, aka individual values, are created in special ways for each of
the four primary types:

| Type      | Value                                                         |
|:----------|:--------------------------------------------------------------|
| Logical   | `TRUE` or `FALSE` (or `T` or `F`)                             |
| Character | surrounded by `"` or `'`. see `?Quotes` for escapes           |
| Double    | decimal (`0.123`), scientific (`1.23e3`), or hexadecimal form |
| Integer   | similar to doubles, ending with `L` (`123L`)                  |

-   There are three special values unique to doubles: `Inf`, `-Inf`, and
    `NaN` (not a number)

#### c()

-   `c()`, short for combine, is used to create longer vectors
-   Determine the vector type with `typeof()`
-   If the inputs to `c()` are other atomic vectors, R will flatten them
    into one atomic vector.

``` r
x1 <- c(1, 2)
x2 <- c(3, 4)
c(x1, x2)
```

    [1] 1 2 3 4

#### Missing values

-   Missing values are represented with `NA`
    -   Each primary type has its own missing value (R usually converts
        to correct type)

| Type      | Missing Value  |
|:----------|:---------------|
| Logical   | `NA`           |
| Character | `NA_character` |
| Double    | `NA_real_`     |
| Integer   | `NA_integer_`  |

-   Most computations involving missing values will return a missing
    value, with a few exceptions:

The 0 power identity

``` r
NA ^ 0
```

    [1] 1

<br>

Boolean logic (or TRUE):

``` r
NA | TRUE
```

    [1] TRUE

<br>

Boolean logic (and FALSE):

``` r
NA & FALSE
```

    [1] FALSE

<br>

-   Use `is.na()` to check for missingness in vectors

``` r
x <- c(NA, 2, 3, NA)
is.na(x)
```

    [1]  TRUE FALSE FALSE  TRUE

#### Testing

The primary types can be checked with the appropriate `is.*()` function:

| Type      | `is.*()` function |
|:----------|:------------------|
| Logical   | `is.logical()`    |
| Character | `is.character()`  |
| Double    | `is.double()`     |
| Integer   | `is.integer()`    |

#### Coercion

Coercion to a different type often happens automatically as a result of
a computation. To deliberately coerce, use the appropriate `as.*()`
function:

| Type      | `as.*()` function |
|:----------|:------------------|
| Logical   | `as.logical()`    |
| Character | `as.character()`  |
| Double    | `as.double()`     |
| Integer   | `as.integer()`    |

Failed coercion generates a warning and returns `NA` for that value.

``` r
as.integer(c("1", "2.5", "bike", "7"))
```

    Warning: NAs introduced by coercion

    [1]  1  2 NA  7

Per `?c`, the hierarchy of types when coercing is NULL \< raw \< logical
\< integer \< double \< complex \< character \< list \< expression

### Exercises

*1. How do you create raw and complex scalars? (See `?raw` and
`?complex`.)*

Raw vectors are created with `raw()`, specifying the single `length`
argument. Raw vectors also have a specific `is.raw()` for checking and
`as.raw()` for coercion.

``` r
r <- raw(2)
r
```

    [1] 00 00

<br>

Complex vectors are created with `complex()` specifying either the
length (via the `length.out` argument), or both the real and imaginary
parts as numeric vectors. <br>

*2. Test your knowledge of the vector coercion rules by predicting the
output of the following uses of `c()`*

Per the type coercion hierarchy, this will be converted to a double,
`c(1, 0)`. 1’s are coerced to `TRUE` (and 0 to `FALSE`), and vice versa.

``` r
x <- c(1, FALSE)
cat(x, " | ", typeof(x))
```

    1 0  |  double

<br>

Coerced to `c("a", "1")` as the character type is determined by the
first value, `"a"`.

``` r
x <- c("a", 1)
cat(x, " | ", typeof(x))
```

    a 1  |  character

<br>

Coerced to `c(1L, 1L)` per the type coercion hierarchy.

``` r
x <- c(TRUE, 1L)
cat(x, " | ", typeof(x))
```

    1 1  |  integer

<br>

*3.1. Why is `1 == "1"` true?*

Per the documentation for `==`, atomic vectors that are of different
types are coerced prior to evaluation. So, the left hand of the
equality, `1`, will be coerced to a character vector prior to
evaluation. Once coerced, the call is `"1" == "1"`, which is clearly
true.

*3.2. Why is `-1 < FALSE` true?*

`<`, like `==` and most operations, also coerce types prior to
evaluation. So after coercion, the call here becomes `-1 < 0`, which is
false.

*3.3. Why is `"one" < 2` false?*

Per the documentation for `<`, string comparison is done at the locale
level. The locale in use can be viewed with `Sys.getlocale()` and
similarly set with `Sys.setlocale()`. Because numbers come before
letters in this sequence, the coerced inequality, `"one" < "2"` is
evaluated as `FALSE`. <br><br>

*4. Why is the default missing value, `NA`, a logical vector? What’s
special about logical vectors? (Hint: think about
`c(FALSE, NA_character_)`.)*

Logical vectors are lowest on the type hierarchy. When they are combined
with another primary type, logicals will always be coerced into the
other type. <br><br>

*5. Precisely what do `is.atomic()`, `is.numeric()`, and `is.vector()`
test for?*

-   `is.atomic()` checks if the vector is of type “logical”, “integer”,
    “numeric”, “complex”, “character” or “raw”
-   `is.numeric()` checks if the vector is a double or integer and *not*
    a factor.
-   `is.vector()` is a generalized `is.*()` function for all vectors.
    The `mode` argument can be specified to check for a specific type,
    or it can be left as `"any"`, the default, to check is the object is
    a vector. `mode` can also be specified as `"numeric"`, running the
    same check as `is.numeric()`.

## 3.3: Attributes

### Notes

#### Getting and setting

-   Attributes are name-value pairs that attach metadata to an R object.
-   Individual attributes can be get and set with `attr()`
-   Attributes en masse are retrieved with `attributes()` and set with
    `structure()`
-   Most attributes, other than **dim** and **names** are lost by most
    operations
    -   To preserve other attributes, create an S3 class, discussed in
        Chapter 13

#### Names

-   Names can be set in three ways:

``` r
# When creating it: 
x <- c(a = 1, b = 2, c = 3)

# By assigning a character vector to names()
x <- 1:3
names(x) <- c("a", "b", "c")

# Inline, with setNames():
x <- setNames(1:3, c("a", "b", "c"))
```

<br>

-   Names should be unique and non-missing, though this is not enforced
    in R

#### Dimensions

-   Adding the **dim** attribute allows a vector to behave like a 2-d
    **matrix** or multi-dimensional **array**
    -   You can also create matrices and arrays with `matrix()` and
        `array()`
-   If a vector has no `dim` attribute set, that is equivalent to a
    vector with `NULL` dimensions
-   Many functions for working with vectors have generalizations for
    working with matrices and arrays

### Exercises

*1. How is `setNames()` implemented? How is `unname()` implemented? Read
the source code.*

``` r
setNames
```

    function (object = nm, nm) 
    {
        names(object) <- nm
        object
    }
    <bytecode: 0x7f8f72c560e8>
    <environment: namespace:stats>

<br>

-   `setNames()` assigns names to the object by calling `names()<-`

``` r
unname
```

    function (obj, force = FALSE) 
    {
        if (!is.null(names(obj))) 
            names(obj) <- NULL
        if (!is.null(dimnames(obj)) && (force || !is.data.frame(obj))) 
            dimnames(obj) <- NULL
        obj
    }
    <bytecode: 0x7f8f82aea1c8>
    <environment: namespace:base>

<br>

-   `unname()` sets names to `NULL` using `names()<-`. If the object is
    a data frame, or `force` is set to `TRUE`, the dimension names are
    set to `NULL` using `dimnames()<-`. <br>

*2.1 What does dim() return when applied to a 1-dimensional vector?*

The dimensions of a 1-d vector are `NULL`, so `dim()` returns `NULL`.

``` r
x <- c(1, 2, 3)
dim(x)
```

    NULL

<br>

*2.2 When might you use NROW() or NCOL()?*

The difference between `NROW()` and `nrow()` (and `NCOL()` and `ncol()`)
is that the capitalized forms return values for one dimensional vectors.

-   `NROW()` returns the length of the vector in the one dimensional
    case
-   `NCOL()` returns `1L` in the one dimensional case <br>\><br>

*3. How would you describe the following three objects? What makes them
different from `1:5`?*

``` r
x1 <- array(1:5, c(1, 1, 5))
x2 <- array(1:5, c(1, 5, 1))
x3 <- array(1:5, c(5, 1, 1))
```

<br>

`x1`, `x2`, and `x3` are all one dimensional arrays in a 3 dimensional
space. Each has a numeric vector of length 3 as the `dim` attribute.
`1:5`, being a 1 dimensional vector, does not have a `dim` attribute -
`dim()` returns `NULL`. <br><br>

*4. An early draft used this code to illustrate `structure()`:*

``` r
structure(1:5, comment = "my attribute")
```

    [1] 1 2 3 4 5

<br>

*But when you print that object you don’t see the comment attribute.
Why? Is the attribute missing, or is there something else special about
it? (Hint: try using help.)*

Checking the documentation for the default method for `print()`,
(`?print.default`), we see that attributes are printed for the object’s
class(es).

``` r
class(1:5)
```

    [1] "integer"

<br>

Since there is no `print` method defined for `integer`, `print.default`
will be used and no attributes printed.

If we wanted the attribute `comment` to be printed with the output, we
would have to create an S3 class with and attribute `comment` and define
a `print` method for that class that includes the `comment` attribute.
<br><br>

## 3.4: S3 atomic vectors

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

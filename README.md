Advanced R - Exercises and Notes
================
Josh Livingston \|
July 8, 2022

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
    syntactically valid names. <br><br>

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

    [1] "0x55f83608e7d0"

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

    [1] "0x55f836189bc0"

<br>

*2. The following code accesses the mean function in multiple ways. Do
they all point to the same underlying function object?*

All accessors to the `mean()` function point to the same object in
memory.

``` r
objs <- list(mean, base::mean, evalq(mean), match.fun("mean"))
obj_addrs(objs)
```

    [1] "0x55f83457eb18" "0x55f83457eb18" "0x55f83457eb18" "0x55f83457eb18"

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

    [1] "0x55f8378ceb48"

<br>

Modifying the object assigned to `y` results in the creation of a new
object.

``` r
y[[3]] <- 4
obj_addr(y)
```

    [1] "0x55f8379a0018"

<br> We see that this is different than the original object’s address

``` r
obj_addr(x) == obj_addr(y)
```

    [1] FALSE

<br> This behavior is called ***copy-on-modify***; i.e., R objects are
immutable – any changes results in the creation of a new object in
memory. <br><br>

#### tracemem()

`base::tracemem()` will track an object’s location in memory.

``` r
x <- c(1, 2, 3)
cat(tracemem(x), "\n")
```

    <0x55f837805698> 

<br> In the example below, a second name, `y` was assigned to an object,
which already had an assigned name `x`. So when `x` or `y` is modified,
copy-on-modify takes place. This will trigger `tracemem()` to print a
memory change.

``` r
y <- x
y[[4]] <- 4L
```

    tracemem[0x55f837805698 -> 0x55f837e5afe8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

<br> `base::untracemem()` is the opposite of `base::tracemem()`

``` r
untracemem(x)
```

<br>

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

    █ [1:0x55f837aff4a8] <list> 
    ├─[2:0x55f8324a0350] <dbl> 
    ├─[3:0x55f8324a0318] <dbl> 
    └─[4:0x55f8324a0200] <dbl> 
     
    █ [5:0x55f83734c9d8] <list> 
    ├─[2:0x55f8324a0350] 
    ├─[3:0x55f8324a0318] 
    └─[6:0x55f8324a02e0] <dbl> 

<br>

#### Data frames

Since data frames are a list of columns, and those columns are vectors,
modifying a column only results in that column being copied:

``` r
d1 <- data.frame(a = c(1, 2, 3), b = c(4, 5, 6))
tracemem(d1)
```

    [1] "<0x55f837929588>"

<br> Here, `tracemem()` shows us that the new column was copied to a new
object in memory.

``` r
d2 <- d1
d2[, 2] <- d2[, 2] * 2
```

    tracemem[0x55f837929588 -> 0x55f836c43d18]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f836c43d18 -> 0x55f836c43998]: [<-.data.frame [<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

<br> And with `lobstr::ref()`, we confirm that both the data.frame
object and the second column were copied.

``` r
ref(d1, d2)
```

    tracemem[0x55f837929588 -> 0x55f837653bd8]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f836c43998 -> 0x55f837b84538]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

    █ [1:0x55f837929588] <df[,2]> 
    ├─a = [2:0x55f837eddd48] <dbl> 
    └─b = [3:0x55f837eddcf8] <dbl> 
     
    █ [4:0x55f836c43998] <df[,2]> 
    ├─a = [2:0x55f837eddd48] 
    └─b = [5:0x55f8378ea278] <dbl> 

<br> Since data.frames are built column-wise, modifying a row results in
copying every column.

``` r
d3 <- d1
d1[1, ] <- d1[1, ] * 2
```

    tracemem[0x55f837929588 -> 0x55f83787fea8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f83787fea8 -> 0x55f83787fde8]: [<-.data.frame [<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

<br>

``` r
untracemem(d1)
ref(d1, d3)
```

    tracemem[0x55f837929588 -> 0x55f8378f6e48]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

    █ [1:0x55f83787fde8] <df[,2]> 
    ├─a = [2:0x55f83734dc48] <dbl> 
    └─b = [3:0x55f83734dbf8] <dbl> 
     
    █ [4:0x55f837929588] <df[,2]> 
    ├─a = [5:0x55f837eddd48] <dbl> 
    └─b = [6:0x55f837eddcf8] <dbl> 

<br>

#### Character vectors

R uses a global string pool in each session. This means that each
element of a character vector points to a string in the globally unique
pool. The references can be viewed in `lobstr::ref()` by setting
`character` to `TRUE`.

``` r
x <- letters[1:3]
ref(x, character = TRUE)
```

    █ [1:0x55f83626d788] <chr> 
    ├─[2:0x55f8322b1a68] <string: "a"> 
    ├─[3:0x55f832603548] <string: "b"> 
    └─[4:0x55f831fb0100] <string: "c"> 

<br>

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

    [1] "<0x55f837b71308>"

``` r
x[[3]] <- 4
```

    tracemem[0x55f837b71308 -> 0x55f837b5f9b8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f837b5f9b8 -> 0x55f837ed4888]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

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

<br>

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

<br>

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

<br>

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

<br>

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

In these exceptions, R executes a modify in place optimization. <br><br>

#### Objects with a single binding

If an object has only one name assigned to it, R will modify in place.

Before:

``` r
a <- c(1, 2, 3)
ref(a)
```

    [1:0x55f8373a5648] <dbl> 

<br> After:

``` r
a[[3]] <- 4
ref(a)
```

    [1:0x55f8379d9c38] <dbl> 

<br> Note that this optimization does *not* apply when modifying a
vector’s length. Here, `z` is assigned to a new object upon “adding” a
fourth element to the vector `z`.

Before:

``` r
z <- letters[1:3]
obj_addr(z)
```

    [1] "0x55f837bede58"

<br>

After:

``` r
z[[4]] <- "d"
obj_addr(z)
```

    [1] "0x55f837917a18"

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

    [1] "<0x55f837e84a08>"

``` r
for (i in seq_along(x)) {
  x[[i]] <- x[[i]] * 5
}
```

    tracemem[0x55f837e84a08 -> 0x55f837bf5ef8]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f837bf5ef8 -> 0x55f837bf5db8]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f837bf5db8 -> 0x55f837bf5d18]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f837bf5d18 -> 0x55f837bf5b88]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f837bf5b88 -> 0x55f837bf5a98]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f837bf5a98 -> 0x55f837bf59a8]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f837bf59a8 -> 0x55f837bf5908]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x55f837bf5908 -> 0x55f837bf5818]: [[<-.data.frame [[<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(x)
```

<br> Then a list:

``` r
l <- as.list(x)
tracemem(l)
```

    [1] "<0x55f833c508f8>"

``` r
for (i in seq_along(l)) {
  l[[i]] <- l[[i]] * 5
}
```

    tracemem[0x55f833c508f8 -> 0x55f836dbc638]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(l)
```

<br>

#### Environments

Environments are always modified-in-place, since existing bindings in
that environment continue to have the same reference. <br><br>

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

    █ [1:0x55f836733658] <list> 

<br> Modified list:

``` r
x[[1]] <- x
ref(x)
```

    █ [1:0x55f83760bd38] <list> 
    └─█ [2:0x55f836733658] <list> 

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

| expression |    min |  median |   itr/sec | mem_alloc |
|:-----------|-------:|--------:|----------:|----------:|
| df         | 91.6µs | 117.5µs |  6865.763 |    99.9KB |
| l          | 14.2µs |  18.1µs | 49036.377 |    78.3KB |

<br> With a 250 x 400 data set, the differences in both speed and memory
allocation are much more pronounced. Here, 70x faster and a quarter of
the memory used.

``` r
df <- as.data.frame(matrix(runif(1e5), ncol = 400))
l <- as.list(df)

bm <- mark(df = mult_five_seq(df), l = mult_five_seq(l))
knitr::kable(bm[, 1:5])
```

| expression |     min |  median |    itr/sec | mem_alloc |
|:-----------|--------:|--------:|-----------:|----------:|
| df         |  13.4ms |  14.3ms |   69.49104 |    3.26MB |
| l          | 280.8µs | 361.8µs | 2671.71058 |  803.17KB |

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
    -   raw <br><br>

#### Scalars

Scalars, aka individual values, are created in special ways for each of
the four primary types:

| Type      | Value                                                         |
|:----------|:--------------------------------------------------------------|
| Logical   | `TRUE` or `FALSE` (or `T` or `F`)                             |
| Character | surrounded by `"` or `'`. see `?Quotes` for escape characters |
| Double    | decimal (`0.123`), scientific (`1.23e3`), or hexadecimal form |
| Integer   | similar to doubles, ending with `L` (`123L`)                  |

-   There are three special values unique to doubles: `Inf`, `-Inf`, and
    `NaN` (not a number) <br><br>

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

<br>

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

<br>

#### Testing

The primary types can be checked with the appropriate `is.*()` function:

| Type      | `is.*()` function |
|:----------|:------------------|
| Logical   | `is.logical()`    |
| Character | `is.character()`  |
| Double    | `is.double()`     |
| Integer   | `is.integer()`    |

<br>

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
<br><br>

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

Complex vectors are created with `complex()`, specifying either the
length (via the `length.out` argument) or both the real and imaginary
parts as numeric vectors. <br><br>

*2. Test your knowledge of the vector coercion rules by predicting the
output of the following uses of `c()`*

Per the type coercion hierarchy, `c(1, FALSE)` will be converted to a
double, `c(1, 0)`. 1’s are coerced to `TRUE` (and 0 to `FALSE`), and
vice versa

``` r
x <- c(1, FALSE)
cat(x, "\n", typeof(x))
```

    1 0 
     double

<br>

`c("a", 1)`, will be coerced to `c("a", "1")` as the character type is
about double in the type coversion hierarchy

``` r
x <- c("a", 1)
cat(x, "\n", typeof(x))
```

    a 1 
     character

<br>

`c(TRUE, 1L)`, will b coerced to `c(1L, 1L)` per the type coercion
hierarchy

``` r
x <- c(TRUE, 1L)
cat(x, "\n", typeof(x))
```

    1 1 
     integer

<br>

*3.1. Why is `1 == "1"` true?*

Per the documentation for `==`, atomic vectors that are of different
types are coerced prior to evaluation. So, the left hand of the
equality, `1`, will be coerced to a character vector prior to
evaluation. Once coerced, the call is `"1" == "1"`, which is clearly
true.

*3.2. Why is `-1 < FALSE` true?*

`<`, like `==`, also coerces types prior to evaluation. So after
coercion, the call here becomes `-1 < 0`, which is true.

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

-   `is.atomic()` checks if an object is of type “logical”, “integer”,
    “numeric”, “complex”, “character” or “raw”
-   `is.numeric()` checks if an object is a double or integer vector and
    *not* a factor.
-   `is.vector()` is a generalized `is.*()` function for all vectors.
    The `mode` argument can be specified to check for a specific type,
    including lists. It can also be left as “any”, the default, to check
    is the object is a vector. `mode` can also be specified as
    “numeric”, running the same check as `is.numeric()`. <br><br>

## 3.3: Attributes

### Notes

#### Getting and setting

-   Attributes are name-value pairs that attach metadata to an R object
-   Individual attributes can be get and set with `attr()`
-   Attributes are retrieved en masse with `attributes()` and set with
    `structure()`
-   Most attributes, other than **dim** and **names** are lost by most
    operations
    -   To define and preserve attributes, create an S3 class, discussed
        in Chapter 13 <br><br>

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
    in R <br><br>

#### Dimensions

-   Adding the **dim** attribute allows a vector to behave like a 2-d
    **matrix** or multi-dimensional **array**
    -   You can also create matrices and arrays with `matrix()` and
        `array()`
-   If a vector has no `dim` attribute set, that is equivalent to a
    vector with `NULL` dimensions
-   Many functions for working with vectors have generalizations for
    working with matrices and arrays <br><br>

### Exercises

*1. How is `setNames()` implemented? How is `unname()` implemented? Read
the source code.*

`setNames()` assigns names to the object by calling `names()<-`

``` r
setNames
```

    function (object = nm, nm) 
    {
        names(object) <- nm
        object
    }
    <bytecode: 0x55f8374b3850>
    <environment: namespace:stats>

<br>

`unname()` sets names to `NULL` using `names()<-`. If the object is a
data frame, or `force` is set to `TRUE`, the dimension names are set to
`NULL` using `dimnames()<-`

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
    <bytecode: 0x55f834cef558>
    <environment: namespace:base>

<br>

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
Both functions return the same value for matrices, arrays, and data
frames.

For one dimensional objects: - `NROW()` returns the length of the
vector - `NCOL()` returns `1L` <br>\><br>

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
`1:5`, being a 1 dimensional vector, does not have a `dim` attribute.

``` r
dim(1:5)
```

    NULL

<br>

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
(`?print.default`), we see that attributes are printed depending on an
object’s class(es).

`1:5` is stored as an “integer”

``` r
class(1:5)
```

    [1] "integer"

<br>

Since there is no `print` method defined for “integer”,
`print.default()` will be used and no attributes printed.

If we wanted the attribute `comment` to be printed with the output, we
would have to create an S3 class with and attribute `comment` and define
a `print` method for that class that includes the `comment` attribute.
<br><br>

## 3.4: S3 atomic vectors

### Notes

Discussed more in Chapter 13, S3 objects have a `class` attribute, which
means it will have special behavior for generic functions.

There are four important S3 vectors in base R:

-   Categorical data in **factor** vectors, an integer
-   Dates in **Date** vectors, a double
-   Date-times in **POSIXct** vectors, a double
-   Durations in **difftime** vectors, a double <br><br>

#### Factors

Factors are useful when looking at categorical data. They sit on top of
integers with two attributes:

-   `class`: “factor”
-   `levels`: defines factor’s categories

Ordered factors are just like factors, except that the order of the
factors is meaningful <br><br>

#### Dates

Dates are built on double vectors with a `class` attribute “Date”.

The value of the double represents the number of days since 1970-01-01.

``` r
d <- as.Date("1971-01-01")
unclass(d)
```

    [1] 365

<br>

#### Date-times

R stores date-time data in two ways, POSIXct, and POSIXlt. POSIX stands
for Portable Operating System Interface, ct for calendar time, and lt
for local time. POSIXct is the simplest usage.

POSIXct variables are built on top of double variables, with two
attributes:

-   `class`: “POSIXct”
-   `tzone`: “UTC”, “GMT”, “” for local, or a [timezone
    name](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

The `tzone` attributes controls the timezone, and can be modified using
`attr()` or `structure()`. See [this Wikipedia
page](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) for
a list of the timezone names and `?timezones` for where to locate the tz
database on your system. <br><br>

#### Durations

Durations, representing the time between two dates or date-times, are
stored in **difftimes**.

Difftimes are built on doubles with a `units` attribute that determines
how the duration should be calculated. <br><br>

### Exercises

*1. What sort of object does `table()` return? What is its type? What
attributes does it have? How does the dimensional change as you tabulate
more variables?*

From viewing its structure with `str()`, we see that `t` is a built on
an integer, appearing to be a 1-dimensional array. It has an additional
attribute `dimnames` containing the factor levels.

``` r
x <- c("cat", "dog", "dog")
f <- factor(x, levels = c("cat", "dog", "horse"))
t <- table(f)
str(t)
```

     'table' int [1:3(1d)] 1 2 0
     - attr(*, "dimnames")=List of 1
      ..$ f: chr [1:3] "cat" "dog" "horse"

<br>

Looking at `dim(t)` we confirm that this is a 1 dimensional array.

``` r
dim(t)
```

    [1] 3

<br>

`attributes()` also provides a clean look at `t`

``` r
attributes(t)
```

    $dim
    [1] 3

    $dimnames
    $dimnames$f
    [1] "cat"   "dog"   "horse"


    $class
    [1] "table"

<br>

For each variable added to the tabulation, the dimensionality increases
by 1. Here with two variables, we note that `t2` is a two dimensional
array.

``` r
y <- c("female", "male", "female")
g <- factor(y)
t2 <- table(f, g)
attributes(t2)
```

    $dim
    [1] 3 2

    $dimnames
    $dimnames$f
    [1] "cat"   "dog"   "horse"

    $dimnames$g
    [1] "female" "male"  


    $class
    [1] "table"

<br>

And `t3`, with three tabulated variables, is a three dimensional array.

``` r
z <- c("black", "black", "brown")
h <- factor(z)
t3 <- table(f, g, h)
attributes(t3)
```

    $dim
    [1] 3 2 2

    $dimnames
    $dimnames$f
    [1] "cat"   "dog"   "horse"

    $dimnames$g
    [1] "female" "male"  

    $dimnames$h
    [1] "black" "brown"


    $class
    [1] "table"

<br>

*2. What happens to a factor when you modify its levels?*

Using `tracemem()`, we see that this is considered a modification, so
copy-on-modify takes place.

``` r
f1 <- factor(letters)
tracemem(f1)
```

    [1] "<0x55f83a30f848>"

``` r
levels(f1) <- rev(levels(f1))
```

    tracemem[0x55f83a30f848 -> 0x55f83a30f008]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

<br>

*3. What does this code do? How do `f2` and `f3` differ from `f1`?*

``` r
f2 <- rev(factor(letters))
f3 <- factor(letters, levels = rev(letters))
```

<br>

The key difference here is the order of the levels in each output. Since
`rev()` reverses the vector’s values, not the values the `levels`
attributes the levels for `f2`, are in alphabetical order:

``` r
levels(f2)[1:5]
```

    [1] "a" "b" "c" "d" "e"

<br>

For `f3`, the levels are in reverse alphabetical order, as `rev()` was
called on the levels themselves when it was created.

``` r
levels(f3)[1:5]
```

    [1] "z" "y" "x" "w" "v"

<br><br>

## 3.5: Lists

### Notes

#### Creating

Lists can be created using `list()`.

-   Lists contain references to other objects
-   A list’s those objects can be of any type, including other lists
-   Because a list can contain another list, lists are sometimes called
    **recursive** vectors
-   `c()` combines multiple lists into one. If some of the inputs to
    `c()` are atomic vectors and others lists, R will coerce the vectors
    to lists prior to combining <br><br>

#### Testing and coercion

-   `typeof()` returns “list” for lists
-   Check if a list with `is.list()` and coerce to a list with
    `as.list()`
-   You can also coerce a list to an atomic vector with `unlist()`
    -   From the book, The rules for the type resulting from `unlist()`
        are *“complex, not well documented, and not always equivalent to
        what you’d get with `c()`”* <br><br>

#### Matrices and arrays

Similarly to atomic vectors, you *can* add a `dim` attribute to lists to
create list-matrices or list-arrays, if you want to. <br><br>

### Exercises

*1. List all the ways that a list differs from an atomic vector.*

-   Because a list contains references, the list’s elements can be of
    any type
-   A list stores references to other objects in memory, while a vector
    only stores one object in memory

Here, the vector is seen occupying one location in memory

``` r
x <- letters
ref(x)
```

    [1:0x55f8322d8d10] <chr> 

<br>

While the list occupies several locations. Note also that `x` and
`letters` are references to the same object in memory

``` r
l <- list(x, LETTERS, letters)
ref(l)
```

    █ [1:0x55f83761d638] <list> 
    ├─[2:0x55f8322d8d10] <chr> 
    ├─[3:0x55f83229e420] <chr> 
    └─[2:0x55f8322d8d10] 

<br>

*2. Why do you need to use `unlist()` to convert a list to an atomic
vector? Why doesn’t `as.vector()` work?*

Since lists are types of vectors, the list itself will be returned when
passed to `as.vector()`. This is confirmed per the documentation at
`?as.vector`. <br><br>

*3. Compare and contrast `c()` and `unlist()` when combining a date and
date-time into a single vector.*

``` r
d <- Sys.Date()
dt <- Sys.time()
```

-   `c()` coerces into a Date or POSIXct variable, whichever is called
    first.

Date first:

``` r
x1 <- c(d, dt)
str(x1)
```

     Date[1:2], format: "2022-07-08" "2022-07-08"

<br>

POSIXct first:

``` r
x2 <- c(dt, d)
str(x2)
```

     POSIXct[1:2], format: "2022-07-08 17:42:38" "2022-07-08 00:00:00"

<br>

-   `unlist()` takes a list and returns the atomic components only, so
    it coerces both elements into a double and returns the resulting
    atomic vector.

``` r
y <- unlist(list(d, dt))
str(y)
```

     num [1:2] 1.92e+04 1.66e+09

<br><br>

## 3.6: Data frames and tibbles

### Notes

Data frames and tibbles are two important S3 vectors built on lists

-   Data frames are a named list of vectors with three attributes:
    -   `names` for column names
    -   `row.names` for row names
    -   `class`, “data.frame”
-   Unlike regular lists, data frames have a requirement that all vector
    elements have the same `NROW()`
    -   Most of the time, this is equivalent to saying all columns must
        have the same length. However, as we’ll see later, data frames
        can be columns, so stating the requirement in terms of `NROW()`
        is necessary to accommodate these cases.
    -   Gives data frames the same properties as matrices
-   Tibbles are a modern “equivalent” to a data.frame, provided by the
    `tibble` package.

``` r
library(tibble)
t <- tibble()
attributes(t)
```

    $class
    [1] "tbl_df"     "tbl"        "data.frame"

    $row.names
    integer(0)

    $names
    character(0)

<br>

#### Creating

-   `data.frame()` to create data frames
-   `tibble::tibble()` to create tibbles

Differences in creation between tibbles and data frames:

-   Tibbles never coerce vectors
    -   A common need is to suppress string to factor coercion in
        `data.frame()` by setting `stringsAsFactors` to `FALSE`
-   Tibbles surround non-syntactic names with `` ` `` rather than
    transforming them
-   Data frames recycle inputs that are an integer multiple of the
    longest vector, while tibbles only recycle vectors that are of
    length 1
-   Tibbles allow you to refer to created variables during construction
    <br><br>

#### Row names

A character vector can be supplied to label the “rows” of a data frame,
in two ways:

-   The `row.names` argument in `data.frame()`
-   By calling `rownames()`

Tibbles do not support row names for three main reasons - Row names are
stored differently from data - They only work when rows can be
identified via a single string - They *must* be unique, so
repetition/resampling results in new row names

Row names can be converted to a column in a tibble using
`rownames_to_column()` or the `rownames` argument in `as_tibble()`
<br><br>

#### Printing

Four main differences between the `print()` output for a data frames and
for tibbles:

-   Tibbles show the first 10 rows and all columns that fit on screen
-   Columns are labelled with its abbreviated type
-   Wide columns are truncated
-   Color is used when supported to (de)emphasize information <br><br>

#### Subsetting

Discussed more in Chapter 4, you can subset data frames and tibbles like
a 1-D list or a 2-D matrix.

Tibbles modify two undesirable properties of data frames:

-   Data frames will return a vector for `df[, x]` if x is a length one
    vector, and a data frame if x is of length \> 1, unless you specify
    `df[, x, drop = FALSE]`
    -   Tibbles always return another tibble when using `[`
    -   Subsetting a single column from a tibble using `[`, however, can
        cause an issue with some legacy code that expects an atomic
        vector when calling `df[, "col"]`. Use `df[["col"]]` to
        unambiguously return the desired column as an atomic vector
        whether subsetting a data frame or tibble

``` r
df1 <- data.frame(xyz = "a")
df2 <- tibble(xyz = "a")
x <- "xyz"
df1[, x]
```

    [1] "a"

``` r
df1[, x, drop = FALSE]
```

      xyz
    1   a

``` r
df2[, x]
```

    # A tibble: 1 × 1
      xyz  
      <chr>
    1 a    

<br>

-   When using `$` data frames will return any variable that starts with
    the input

``` r
str(df1$x)
```

     chr "a"

<br>

Tibbles only return exact matches with `$`

``` r
str(df2$x)
```

    Warning: Unknown or uninitialised column: `x`.

     NULL

<br>

#### Testing and coercing

-   Check with `is.data.frame()` or `is_tibble()`
-   Coerce with `as.data.frame()` or `as_tibble()` <br><br>

#### List columns

Since data frames are lists of vectors, a data frame can have a column
as a list.

Adding a list column involves an extra step in data frames:

``` r
# Either after the data frame is created
d <- data.frame(a = 1:3)
d$b <- list(4:6)
```

<br>

Lists are fully supported in tibbles

``` r
d <- tibble(
  a = 1:3,
  b = list(4:6)
)
```

<br>

#### Matrix and data frame columns

Extending the length requirement for data frames (that all columns must
be of the same length), it’s actually the `NROW()` of each column that
needs to match. Because of this, data frames and matrices can be
included as columns in a data frame.

Just as with list columns, it must be added after creation or wrapped in
`I()`. Note that wrapping it in `I()` adds a `class` “AsIs”

``` r
d0 <- data.frame(a = 1:3, b = 4:6)
d <- data.frame(x = -2:0, y = I(d0))
str(d)
```

    'data.frame':   3 obs. of  2 variables:
     $ x: int  -2 -1 0
     $ y:Classes 'AsIs' and 'data.frame':   3 obs. of  2 variables:
      ..$ a: int  1 2 3
      ..$ b: int  4 5 6

<br>

Many functions that work with columns assume that all columns are
vectors, so use with caution. <br><br>

### Exercises

*1. Can you have a data frame with zero rows? What about zero columns?*

You can create an empty data frame with zero rows and zero columns.

``` r
d <- data.frame()
str(d)
```

    'data.frame':   0 obs. of  0 variables

<br>

You can add an empty row, but its value is inaccessible

``` r
d[1, ] <- 1L
d[1, ]
```

    data frame with 0 columns and 1 row

<br>

Same outcome for row without a column in a tibble

``` r
d <- tibble()
d[1, ] <- 1L
d[1, ]
```

    # A tibble: 1 × 0

<br>

You can add an empty columns during or after creation

``` r
d <- data.frame(x = character())
d$y <- vector("list")
str(d)
```

    'data.frame':   0 obs. of  2 variables:
     $ x: chr 
     $ y: list()

<br>

*2. What happens if you attempt to set rownames that are not unique?*

R will throw an error

``` r
data.frame(a = 1:3, row.names = rep("x", 3))
```

    Error in data.frame(a = 1:3, row.names = rep("x", 3)) :
      duplicate row.names: x
    NULL

<br>

*3. If `df` is a data frame, what can you say about `t(df)`, and
`t(t(df))`? Perform some experiments, making sure to try different
column types.*

If `df` can behave like a matrix, `t()` will operate as expected

``` r
d <- data.frame(a = 1:2, b = 3:4)
t(d)
```

      [,1] [,2]
    a    1    2
    b    3    4

``` r
t(t(d))
```

         a b
    [1,] 1 3
    [2,] 2 4

<br>

Prior to transposing, `t()` coerces `df` to a matrix. Non-atomic vectors
are coerced by `as.vector()`. For lists, `as.vector()` returns the list,
so transposition occurs at the list-element level.

Looking at the matrix object created as a first step, we see that the
matrix preserved all variable types when coercing a list column into a
matrix column. Remember that matrices are two dimensional vectors

``` r
d <- data.frame(a = 1:3, b = I(list("4", 5, 6L)))
str(as.matrix(d))
```

    List of 6
     $ : int 1
     $ : int 2
     $ : int 3
     $ : chr "4"
     $ : num 5
     $ : int 6
     - attr(*, "dim")= int [1:2] 3 2
     - attr(*, "dimnames")=List of 2
      ..$ : NULL
      ..$ : chr [1:2] "a" "b"

<br>

So `t()` works as “expected” with list columns

``` r
t(d)
```

      [,1] [,2] [,3]
    a 1    2    3   
    b "4"  5    6   

``` r
t(t(d))
```

         a b  
    [1,] 1 "4"
    [2,] 2 5  
    [3,] 3 6  

<br>

For data frame columns, `as.matrix()` will combine the data frame
columns (and any nested data frame columns) with the containing data
frame.

``` r
d <- data.frame(a = 1:3, b = 4:6)
d$c <- data.frame(x = 101:103, y = 104:106)
d0 <- data.frame(z = 1001:1003)
d0$zz <- data.frame(z0 = 1:3, z1 = 4:6)
d$d <- d0
as.matrix(d)
```

         a b c.x c.y  d.z d.zz.z0 d.zz.z1
    [1,] 1 4 101 104 1001       1       4
    [2,] 2 5 102 105 1002       2       5
    [3,] 3 6 103 106 1003       3       6

<br>

`t()` works with data frame columns normally after combining

``` r
t(d)
```

            [,1] [,2] [,3]
    a          1    2    3
    b          4    5    6
    c.x      101  102  103
    c.y      104  105  106
    d.z     1001 1002 1003
    d.zz.z0    1    2    3
    d.zz.z1    4    5    6

``` r
t(t(d))
```

         a b c.x c.y  d.z d.zz.z0 d.zz.z1
    [1,] 1 4 101 104 1001       1       4
    [2,] 2 5 102 105 1002       2       5
    [3,] 3 6 103 106 1003       3       6

<br>

*4. What does `as.matrix()` do when applied to a data frame with columns
of different types? How does it differ from `data.matrix()`?*

The behavior for `as.matrix()` is described in answer 3.

`data.matrix()` converts all variables in a data frame to numeric via
`as.numeric()` prior to combining them, so I would expect odd behavior
when converting non numeric variable types

Logical and factor columns are coerced to integers. Character columns
are converted first to factors, then to integers

``` r
a <- letters[1:3]
as.integer(as.factor(a))
```

    [1] 1 2 3

<br>

So `data.matrix()` will use the resulting integer column for the
resulting matrix

``` r
b <- c(TRUE, TRUE, FALSE)
c <- factor(LETTERS[1:3])
d <- data.frame(a, b, c)
data.matrix(d)
```

         a b c
    [1,] 1 1 1
    [2,] 2 1 2
    [3,] 3 0 3

<br>

Any non numeric column that is *not* a logical, factor, or character
column is converted to a numeric column via `as.numeric()`.
`as.numeric()` only works on atomic vectors, so it will convert each
list element at the atomic level

``` r
b <- list(a = 1, b = "2", c = 3L)
d <- data.frame(b = b)
data.matrix(d)
```

         b.a b.b b.c
    [1,]   1   1   3

<br>

Similar application for data frames, being lists themselves

``` r
c <- data.frame(c1 = 4:6, c2 = 7:9)
d <- data.frame(a = 1:3, c = c)
data.matrix(d)
```

         a c.c1 c.c2
    [1,] 1    4    7
    [2,] 2    5    8
    [3,] 3    6    9

<br><br>

## 3.7: NULL

### Notes

-   `NULL` is a data structure with a unique type, “NULL”

``` r
typeof(NULL)
```

    [1] "NULL"

<br>

-   `NULL` is always length zero

``` r
length(NULL)
```

    [1] 0

<br>

-   `NULL` cannot have any attributes

``` r
x <- NULL
attr(x, a) <- 1L
```

    Error in attr(x, a) <- 1L : attempt to set an attribute on NULL
    NULL

<br>

Two common uses of `NULL`:

-   To represent an empty vector
-   To represent an absent vector
    -   `NULL` is often used as a default function value, to signify
        that that value is not needed in the function.
    -   `NA`, in contrast, signifies that the element of a vector is
        absent, not the vector itself <br><br>

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

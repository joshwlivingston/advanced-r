Advanced R - Exercises and Notes
================
Josh Livingston \|
June 27, 2022

<p>This repository stores all of my notes and exercise work-throughs from <a href="https://adv-r.hadley.nz/">Advanced R</a>. This uses the 2nd edition of Hadley Wickham's book.</p>
<p>Source code is organized at the chapter-section level. In each section, notes appear first, followed by the exercises. Exercise question text are written in <em>italics</em>.</p>
<p>This README was rendered using <a href="https::quarto.org">quarto</a>.</p>


# Chapter 2 - Names and values

## 2.2 - Binding basics

### Notes

-   A value does not have a name; a name has a value.
-   That is, a name gets bound to a value as a reference to that value.
-   Assigning a second name to that object does not change the object or
    copy it. It simply assigns a new name as reference to that object.
-   Names have to be pretty. See `?make.names` for rules governing
    syntactically valid names.

### Exercises

<em>1. Explain the relationship between `a`, `b`, `c`, and `d` in the
following code:</em>

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

    [1] "0x7fe6186b02d8"

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

    [1] "0x7fe60f7aa658"

<br>

<em>2. The following code accesses the mean function in multiple ways.
Do they all point to the same underlying function object?</em>

All accessors to the `mean()` function point to the same object in
memory.

``` r
objs <- list(mean, base::mean, evalq(mean), match.fun("mean"))
obj_addrs(objs)
```

    [1] "0x7fe5ff5fd860" "0x7fe5ff5fd860" "0x7fe5ff5fd860" "0x7fe5ff5fd860"

<br>

<em>3. By default, base R data import functions, like `read.csv()`, will
automatically convert non-syntactc names to syntactic ones. Why might
this be problematic? What option allows you to supporess this
behavior?</em>

Column names often represent data, so renaming with `make.names` changes
underlying data. You can suppress this with `check.names = FALSE`.
<br><br>

<em>4. What rules does `make.names()` use to convert non-syntactic names
into syntactic ones?</em>

From the docs, <em>“A syntactically valid name consists of letters,
numbers and the dot or underline characters and starts with a letter or
the dot not followed by a number.”</em> Letters are defined by locale,
but only ASCII digits are used. Invalid characters are translated to
“.”. Missing is translated to “NA”. And reserved words have a “.”
appended to them. Then, values are de-duplicated using `make.unique()`.
<br><br>

<em>5. I slightly simplified the rules that govern syntactic names. Why
is `.123e1` not a syntactic name?</em>

Syntactic names may start with a letter, or a dot not followed by a
number. `.123e1` starts with `.1`, so it is not a syntactically valid
name. <br><br>

## 2.3 - Copy-on-modify

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

    [1] "0x7fe5ea200d98"

<br>

Modiyfing the object assigned to `y` results in the creation of a new
object.

``` r
y[[3]] <- 4
obj_addr(y)
```

    [1] "0x7fe5e99211a8"

We see that this is different than the original object’s address

``` r
obj_addr(x) == obj_addr(y)
```

    [1] FALSE

This behavior is called <em>copy-on-modify</em>; i.e., R objects are
immutable – any changes results in the creation of a new object in
memory.

Copy-on-modify also applies when an object with one standalone reference
is modified. Here, `z` is assigned to a new object upon modification of
the original `z`. We see this by looking at the location in memory
before and after the modificaiton.

Before:

``` r
z <- letters[1:3]
obj_addr(z)
```

    [1] "0x7fe5df3e17f8"

<br>

After:

``` r
z[[4]] <- "d"
obj_addr(z)
```

    [1] "0x7fe60fc67598"

<em>tracemem()

`base::tracemem()` will track an object’s location in memory.

``` r
x <- c(1, 2, 3)
cat(tracemem(x), "\n")
```

    <0x7fe5e82b4638> 

In the example below, a second name, `y` was assigned to an object,
which already had an assigned name `x`. So when `x` or `y` is modified,
copy-on-modify takes place. This will trigger `tracemem()` to print a
memory change.

``` r
y <- x
y[[4]] <- 4L
```

    tracemem[0x7fe5e82b4638 -> 0x7fe5e89b2598]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

`base::untracemem()` is the opposite of `base::tracemem()`

``` r
untracemem(x)
```

<em>Lists

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

    █ [1:0x7fe5e93a9978] <list> 
    ├─[2:0x7fe5e8cbec08] <dbl> 
    ├─[3:0x7fe5e8cbebd0] <dbl> 
    └─[4:0x7fe5e8cbeab8] <dbl> 
     
    █ [5:0x7fe5e8de90a8] <list> 
    ├─[2:0x7fe5e8cbec08] 
    ├─[3:0x7fe5e8cbebd0] 
    └─[6:0x7fe5e8cbeb98] <dbl> 

<em>Dataframes

Since dataframes are a list of columns, and those columns are vectors,
modifiying a column only results in that column being copied:

``` r
d1 <- data.frame(a = c(1, 2, 3), b = c(4, 5, 6))
tracemem(d1)
```

    [1] "<0x7fe60fbf6708>"

Here, `tracemem()` shows us that the new column was copied to a new
object in memory.

``` r
d2 <- d1
d2[, 2] <- d2[, 2] * 2
```

    tracemem[0x7fe60fbf6708 -> 0x7fe60fa61788]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fe60fa61788 -> 0x7fe60fa618c8]: [<-.data.frame [<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

And with `lobstr::ref()`, we confirm that both the data.frame object and
the second column were copied.

``` r
ref(d1, d2)
```

    tracemem[0x7fe60fbf6708 -> 0x7fe60f0b2908]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fe60fa618c8 -> 0x7fe5e8302d48]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

    █ [1:0x7fe60fbf6708] <df[,2]> 
    ├─a = [2:0x7fe5e95cc248] <dbl> 
    └─b = [3:0x7fe5e95cc1f8] <dbl> 
     
    █ [4:0x7fe60fa618c8] <df[,2]> 
    ├─a = [2:0x7fe5e95cc248] 
    └─b = [5:0x7fe60f0cdec8] <dbl> 

Since data.frames are built column-wise, modifying a row results in
copying every column.

``` r
d3 <- d1
d1[1, ] <- d1[1, ] * 2
```

    tracemem[0x7fe60fbf6708 -> 0x7fe5df6e2a08]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fe5df6e2a08 -> 0x7fe5df6e2948]: [<-.data.frame [<- eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(d1)
ref(d1, d3)
```

    tracemem[0x7fe60fbf6708 -> 0x7fe60fc60a88]: FUN lapply ref eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

    █ [1:0x7fe5df6e2948] <df[,2]> 
    ├─a = [2:0x7fe5ea2006b8] <dbl> 
    └─b = [3:0x7fe5ea200668] <dbl> 
     
    █ [4:0x7fe60fbf6708] <df[,2]> 
    ├─a = [5:0x7fe5e95cc248] <dbl> 
    └─b = [6:0x7fe5e95cc1f8] <dbl> 

<em>Character vectors

R uses a global string pool in each session. This means that each
element of a character vector points to a string in the globally unique
pool. The references can be viewed in `lobstr::ref()` by setting
`character` to `TRUE`.

``` r
x <- letters[1:3]
ref(x, character = TRUE)
```

    █ [1:0x7fe60fccd038] <chr> 
    ├─[2:0x7fe5ff1c70e8] <string: "a"> 
    ├─[3:0x7fe60f1acce8] <string: "b"> 
    └─[4:0x7fe60f00e0c0] <string: "c"> 

### Exercises

<em>1. Why is `tracemem(1:10)` not useful?</em>

`1:10` is a sequence no name assigned to it, therefore will not be
traceable after this initial call.

<em>2. Explain why `tracemem()` shows two copies when you run this code.
Hint: carefully look at the difference between this code and the code
shown earlier in the section.</em>

``` r
x <- c(1L, 2L, 3L)
tracemem(x)
```

    [1] "<0x7fe5ea122b08>"

``` r
x[[3]] <- 4
```

    tracemem[0x7fe5ea122b08 -> 0x7fe5ea146288]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 
    tracemem[0x7fe5ea146288 -> 0x7fe5e84e8508]: eval eval eval_with_user_handlers withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir in_input_dir eng_r block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous> execute .main 

``` r
untracemem(x)
```

In this example, the original object `x` is an integer vector. When the
third element of the vector is modifed to 4, a double, the vector is
first modified by being converted to a double vector. Then it is
modified again when the third element is modified. This results in two
copies-on-modify, reflected in the `tracemem()` output.

# Chapter 3 - Vectors

# Chapter 4 - Subsetting

# Chapter 5 - Control Flow

# Chapter 6 - Functions

# Chapter 7 - Environments

# Chapter 8 - Conditions

# Chapter 9 - Functionals

# Chapter 10 - Function factories

# Chapter 11 - Function operators

# Chapter 12 - Base types

# Chapter 13 - S3

# Chapter 14 - R6

# Chapter 15 - S4

# Chapter 16 - Trade-offs

# Chapter 17 - Big picture

# Chapter 18 - Expressions

# Chapter 19 - Quasiquotation

# Chapter 20 - Evaluation

# Chapter 21 - Translating R code

# Chapter 22 - Debugging

# Chapter 23 - Measuring performance

# Chapter 24 - Improving performance

# Chapter 25 - Rewriting R code in C++

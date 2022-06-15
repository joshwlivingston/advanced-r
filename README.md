Advanced R - Exercises and Notes
================
Josh Livingston \|
June 15, 2022

<p>This repository stores all of my notes and exercise work-throughs from <a href="https://adv-r.hadley.nz/">Advanced R</a>. This uses the 2nd edition of Hadley Wickham's book.</p>
<p>Source code is organized at the chapter-section level. In each section, notes appear before the exercises. Exercise text is noted with <em><b>bold and italicized text</b></em>.</p>
<p>This README was rendered using <a href="https::quarto.org">quarto</a>.</p>
<h2>Table of Contents</h2>


-   <a href="#i---foundations" id="toc-i---foundations">I - Foundations</a>
    -   <a href="#chapter-2---names-and-values"
        id="toc-chapter-2---names-and-values">Chapter 2 - Names and values</a>
        -   <a href="#binding-basics" id="toc-binding-basics">2.2 - Binding
            basics</a>
        -   <a href="#copy-on-modify" id="toc-copy-on-modify">2.3 -
            Copy-on-modify</a>
    -   <a href="#chapter-3---vectors" id="toc-chapter-3---vectors">Chapter 3 -
        Vectors</a>
    -   <a href="#chapter-4---subsetting"
        id="toc-chapter-4---subsetting">Chapter 4 - Subsetting</a>
    -   <a href="#chapter-5---control-flow"
        id="toc-chapter-5---control-flow">Chapter 5 - Control Flow</a>
    -   <a href="#chapter-6---functions" id="toc-chapter-6---functions">Chapter
        6 - Functions</a>
    -   <a href="#chapter-7---environments"
        id="toc-chapter-7---environments">Chapter 7 - Environments</a>
    -   <a href="#chapter-8---conditions"
        id="toc-chapter-8---conditions">Chapter 8 - Conditions</a>
-   <a href="#ii---functional-programming"
    id="toc-ii---functional-programming">II - Functional programming</a>
    -   <a href="#chapter-9---functionals"
        id="toc-chapter-9---functionals">Chapter 9 - Functionals</a>
    -   <a href="#chapter-10---function-factories"
        id="toc-chapter-10---function-factories">Chapter 10 - Function
        factories</a>
    -   <a href="#chapter-11---function-operators"
        id="toc-chapter-11---function-operators">Chapter 11 - Function
        operators</a>
-   <a href="#iii-object-oriented-programming"
    id="toc-iii-object-oriented-programming">III Object-oriented
    programming</a>
    -   <a href="#chapter-12---base-types"
        id="toc-chapter-12---base-types">Chapter 12 - Base types</a>
    -   <a href="#chapter-13---s3" id="toc-chapter-13---s3">Chapter 13 - S3</a>
    -   <a href="#chapter-14---r6" id="toc-chapter-14---r6">Chapter 14 - R6</a>
    -   <a href="#chapter-15---s4" id="toc-chapter-15---s4">Chapter 15 - S4</a>
    -   <a href="#chapter-16---trade-offs"
        id="toc-chapter-16---trade-offs">Chapter 16 - Trade-offs</a>
-   <a href="#iv---metaprogramming" id="toc-iv---metaprogramming">IV -
    Metaprogramming</a>
    -   <a href="#chapter-17---big-picture"
        id="toc-chapter-17---big-picture">Chapter 17 - Big picture</a>
    -   <a href="#chapter-18---expressions"
        id="toc-chapter-18---expressions">Chapter 18 - Expressions</a>
    -   <a href="#chapter-19---quasiquotation"
        id="toc-chapter-19---quasiquotation">Chapter 19 - Quasiquotation</a>
    -   <a href="#chapter-20---evaluation"
        id="toc-chapter-20---evaluation">Chapter 20 - Evaluation</a>
    -   <a href="#chapter-21---translating-r-code"
        id="toc-chapter-21---translating-r-code">Chapter 21 - Translating R
        code</a>
-   <a href="#v---techniques" id="toc-v---techniques">V - Techniques</a>
    -   <a href="#chapter-22---debugging"
        id="toc-chapter-22---debugging">Chapter 22 - Debugging</a>
    -   <a href="#chapter-23---measuring-performance"
        id="toc-chapter-23---measuring-performance">Chapter 23 - Measuring
        performance</a>
    -   <a href="#chapter-24---improving-performance"
        id="toc-chapter-24---improving-performance">Chapter 24 - Improving
        performance</a>
    -   <a href="#chapter-25---rewriting-r-code-in-c"
        id="toc-chapter-25---rewriting-r-code-in-c">Chapter 25 - Rewriting R
        code in C++</a>

## I - Foundations

### Chapter 2 - Names and values

#### 2.2 - Binding basics

-   A value does not have a name; a name has a value.
-   That is, a name gets bound to a value as a reference to that value.
-   Assigning a second name to that object does not change the object or
    copy it. It simply assigns a new name as reference to that object.
-   Names have to be pretty. See `?make.names` for rules governing
    syntactically valid names.

<b><em>1. Explain the relationship between `a`, `b`, `c`, and `d` in the
following code:</em></b>

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

    [1] "0x7fedd6168a58"

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

    [1] "0x7fede670c280"

<br>

<b><em>2. The following code accesses the mean function in multiple
ways. Do they all point to the same underlying function object?</em></b>

All accessors to the `mean()` function point to the same object in
memory.

``` r
objs <- list(mean, base::mean, evalq(mean), match.fun("mean"))
obj_addrs(objs)
```

    [1] "0x7fedb647ee60" "0x7fedb647ee60" "0x7fedb647ee60" "0x7fedb647ee60"

<br>

<b><em>3. By default, base R data import functions, like `read.csv()`,
will automatically convert non-syntactc names to syntactic ones. Why
might this be problematic? What option allows you to supporess this
behavior?</em></b>

Column names often represent data, so renaming with `make.names` changes
underlying data. You can suppress this with `check.names = FALSE`.
<br><br>

<b><em>4. What rules does `make.names()` use to convert non-syntactic
names into syntactic ones?</em></b>

From the docs, <em>“A syntactically valid name consists of letters,
numbers and the dot or underline characters and starts with a letter or
the dot not followed by a number.”</em> Letters are defined by locale,
but only ASCII digits are used. Invalid characters are translated to
“.”. Missing is translated to “NA”. And reserved words have a “.”
appended to them. Then, values are de-duplicated using `make.unique()`.
<br><br>

<b><em>5. I slightly simplified the rules that govern syntactic names.
Why is `.123e1` not a syntactic name?</em></b>

Syntactic names may start with a letter, or a dot not followed by a
number. `.123e1` starts with `.1`, so it is not a syntactically valid
name. <br><br>

#### 2.3 - Copy-on-modify

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

    [1] "0x7fedc1449d38"

<br>

Modiyfing the object assigned to `y` results in the creation of a new
object.

``` r
y[[3]] <- 4
obj_addr(y)
```

    [1] "0x7fede64b9658"

We see that this is different than the original object’s address

``` r
obj_addr(x) == obj_addr(y)
```

    [1] FALSE

This behavior is called <em>copy-on-modify</em>; i.e., R objects are
immutable – any changes results in the creation of a new object in
memory.

### Chapter 3 - Vectors

### Chapter 4 - Subsetting

### Chapter 5 - Control Flow

### Chapter 6 - Functions

### Chapter 7 - Environments

### Chapter 8 - Conditions

## II - Functional programming

### Chapter 9 - Functionals

### Chapter 10 - Function factories

### Chapter 11 - Function operators

## III Object-oriented programming

### Chapter 12 - Base types

### Chapter 13 - S3

### Chapter 14 - R6

### Chapter 15 - S4

### Chapter 16 - Trade-offs

## IV - Metaprogramming

### Chapter 17 - Big picture

### Chapter 18 - Expressions

### Chapter 19 - Quasiquotation

### Chapter 20 - Evaluation

### Chapter 21 - Translating R code

## V - Techniques

### Chapter 22 - Debugging

### Chapter 23 - Measuring performance

### Chapter 24 - Improving performance

### Chapter 25 - Rewriting R code in C++

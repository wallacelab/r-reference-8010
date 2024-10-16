---
title: "R Basics"
author: "Jason G. Wallace"
date: "2020-09-29"
output: html_document
---




# R Basics
This page is for the basic nuts-and-bolts functions that work in R. This is not meant to be comprehensive. If you want in-depth tutorials on R basics, please try:

* [Data Camp](https://www.datacamp.com/courses/free-introduction-to-r) - Interactive tutorials
* [Quick-R](https://www.statmethods.net/r-tutorial) - Tutorial and basic help guide
* [R for Data Science](http://r4ds.had.co.nz) - Free book on using R for data science

## Data types
R has 4 basic data types we deal with:

* **integer** - Whole numbers
* **numeric** - Decimal numbers
* **logical** - True or False
* **character** - Character strings

We also work with **factors**, which are a sort of indexed character vector (so "Jane" = 1, "Bert" = 2, etc.) These are most useful when you have something that can take a limited set of values, such as treatments in your experiments.

## Vectors

### c()
c() (for 'combine') is the most basic vector function. It is what you use to make vectors.


```r
> # Make a vector of number values
> c(1.6, 2.5, 10-0, -9, 14)
## [1]  1.6  2.5 10.0 -9.0 14.0
> 
> # Make a vector of character strings
> c("Alpha", "Beta", "Charlie", "Delta")
## [1] "Alpha"   "Beta"    "Charlie" "Delta"
> 
> # Make a character of logical values
> c(TRUE, TRUE, FALSE, FALSE, TRUE)
## [1]  TRUE  TRUE FALSE FALSE  TRUE
```

You can use a colon (":") to quickly make a vector consisting of all intergers between two values. 

```r
> 1:8
## [1] 1 2 3 4 5 6 7 8
> -2:2
## [1] -2 -1  0  1  2
```

### length()
The length() function tells you how long a vector is


```r
> x = c(1,2,3,4,5)
> length(x)
## [1] 5
> y = 1:259
> length(y)
## [1] 259
```

### class()
The class() function tells you what type of vector you're working with.


```r
> class(1:8) # An integer vector
## [1] "integer"
> class(c("A","B","C")) # A character vector
## [1] "character"
> class(c(TRUE, FALSE)) # Logical vector
## [1] "logical"
```

It also works on other, more complex data types


```r
> # Matrix
> m = matrix(1:9, nrow=3)
> class(m)
## [1] "matrix"
> # Data frame
> d = data.frame(a=1:5, b=6:10)
> class(d)
## [1] "data.frame"
```

### as.____()

The as.____() family of functions will try to force your data into a certain type. If it can't, it gets set to missing (NA)


```r
> x = c("1.2", "3", "Alpha", "TRUE", "0") # A character vector
> as.integer(x)
## Warning: NAs introduced by coercion
## [1]  1  3 NA NA  0
> as.numeric(x)
## Warning: NAs introduced by coercion
## [1] 1.2 3.0  NA  NA 0.0
> as.logical(x)
## [1]   NA   NA   NA TRUE   NA
> as.factor(x)
## [1] 1.2   3     Alpha TRUE  0    
## Levels: 0 1.2 3 Alpha TRUE
```


## Matrices
### matrix()
You make a matrix by using the matrix() function It requires the data itself and the number of rows and/or columns. 

```r
> matrix(1:4, nrow=1) # Infer a 1x4 matrix
##      [,1] [,2] [,3] [,4]
## [1,]    1    2    3    4
> matrix(1:4, nrow=2) # Infer a 2x2 matrix
##      [,1] [,2]
## [1,]    1    3
## [2,]    2    4
> matrix(1:4, nrow=2, ncol=4) # Force a 2x4 matrix by recycling the data
##      [,1] [,2] [,3] [,4]
## [1,]    1    3    1    3
## [2,]    2    4    2    4
> matrix(1:4, nrow=2, ncol=4, byrow=TRUE) # Fill across rows instead of down columns)
##      [,1] [,2] [,3] [,4]
## [1,]    1    2    3    4
## [2,]    1    2    3    4
> matrix(1:4, nrow=1, ncol=5) # Throws an warning because can't recycle the input data an even number of time
## Warning in matrix(1:4, nrow = 1, ncol = 5): data length [4] is not a sub-
## multiple or multiple of the number of columns [5]
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    1    2    3    4    1
```


### Matrix dimensions: dim(), nrow() and ncol() 
These are just quick functions to get the size of a matrix or data frame.

```r
> mymatrix = matrix(1:8, nrow=2) # 2x4 matrix
> mymatrix
##      [,1] [,2] [,3] [,4]
## [1,]    1    3    5    7
## [2,]    2    4    6    8
> nrow(mymatrix) # 2 rows
## [1] 2
> ncol(mymatrix) # 4 columns
## [1] 4
> dim(mymatrix) # 2x4 matrix
## [1] 2 4
```



## Data frames
### data.frame()
You make a data frame with the data.frame() function, where each argument is a column. If you name the arguments, those become the column names. Note that strings are coerced into factors unless you set stringsAsFactors to FALSE.

```r
> df = data.frame(number=1:5, letter=LETTERS[1:5], isred=c(T,F,F,T,F)) # T and F are shortcuts for TRUE and FALSE
> df
##   number letter isred
## 1      1      A  TRUE
## 2      2      B FALSE
## 3      3      C FALSE
## 4      4      D  TRUE
## 5      5      E FALSE
```

### dim(), nrow() and ncol()
These functions work exactly the same as with matrices, above.

### names()
The names() function returns the column names of a data frame.

```r
> names(df)
## [1] "number" "letter" "isred"
```


## Lists
### list()
A list is like a data frame in that it can store multiple objects of different types, but they don't have to be the same length. Lists can store just about anything, including data frames, other lists, and even functions. length() will give you the number of items in a list. Items inside a list can be accessed with the $ operator or giving the list index in double brackets [[]]

```r
> mylist = list(a=1:5, b=c(T,T,F), c=mean, d=list(a=1, b=2:5))
> mylist
## $a
## [1] 1 2 3 4 5
## 
## $b
## [1]  TRUE  TRUE FALSE
## 
## $c
## function (x, ...) 
## UseMethod("mean")
## <bytecode: 0x26b86b0>
## <environment: namespace:base>
## 
## $d
## $d$a
## [1] 1
## 
## $d$b
## [1] 2 3 4 5
> length(mylist)
## [1] 4
> length(mylist$b) # Length of item 'b' inside the list
## [1] 3
> length(mylist[[2]]) # Same as above, since b is item 2
## [1] 3
```


## Reading and writing data
### scan()
scan() is the most basic function to read data in. The data must be all of the same type, and you must tell R what that type is by giving it an example. (This is usually done by creating a 0-length vector of the appropriate type, such as character(), integer(), etc.) scan() will treat any whitespace as a separator by default, but you can change this with the 'sep' argument.

```r
> scan('data/demo_scan.txt', what=character()) # Read as character
## [1] "1.0"   "2.5"   "37.6"  "-10.5" "14"    "7"
> 
> scan('data/demo_scan.txt', what=numeric()) # Read as numeric (default)
## [1]   1.0   2.5  37.6 -10.5  14.0   7.0
```

### read.table() and read.csv()
read.table() is the generic function to read in a dataframe. It has a lot of arguments you can tweak depending on the input format. read.csv() is a wrapper function that just sets most of these to be the most common for comma-separated documents.

```r
> read.table("data/demo_read_table.txt") # space-separated
##     V1          V2    V3
## 1 plot     variety yield
## 2  001 GoldenQueen  14.3
## 3  002   Silverado  16.3
## 4  003     Bicolor  15.0
## 5  004   Bodacious  15.1
> 
> read.table("data/demo_read_table.csv") # same, but comma-separated
##                     V1
## 1   plot,variety,yield
## 2 001,GoldenQueen,14.3
## 3   002,Silverado,16.3
## 4     003,Bicolor,15.0
## 5   004,Bodacious,15.1
```


### write.table() and write.csv()
These are like read.table() and read.csv(), but in reverse: they write a dataframe to a file.

```r
> df = data.frame(col1=1:10, col2=LETTERS[1:10])
> write.csv(df, file="data/demo_write_csv.csv")
```

## Boolean expressions

Boolean expressions have to do with whether something is TRUE or FALSE. They are very useful for checking our data and especially for filtering out things we don't want.

### Basic comparison operators

The basic comparison operators are:

* `>` Greater than
* `>=` Greater than or equal to
* `<` Less than
* `<` Less than or equal to
* `==` Is equal to
* `!=` Is not equal to


```r
> x = c(1, 2, 3, 4)
> 
> # Which values are greater than 2
> x > 2  
## [1] FALSE FALSE  TRUE  TRUE
> 
> # Same, but now greater-than-or-equal-to
> x >= 2 
## [1] FALSE  TRUE  TRUE  TRUE
> 
> # Testing equality
> x == 3
## [1] FALSE FALSE  TRUE FALSE
> x != 1
## [1] FALSE  TRUE  TRUE  TRUE
```
### Missing and non-finite values

Missing and non-finite values (NULL, Inf, -Inf) cause issues with normal comparison operators, so there is a special family of functions to deal with them

* is.na() tests if something is missing
* is.finite() tests if something is a finite value


```r
> x = c(2, NA, Inf)
> 
> # Missing values propagate
> x > 2 
## [1] FALSE    NA  TRUE
> 
> # Doesn't actually test for missing; instead everything is missing
> x == NA  
## [1] NA NA NA
> 
> # How to actually test what is missing
> is.na(x) 
## [1] FALSE  TRUE FALSE
> 
> # Tests for finite values
> is.finite(x) 
## [1]  TRUE FALSE FALSE
```
### Boolean operators
Often we want to combine Boolean expressions to test based on multiple things at once. We do this with the three Boolean operators:
* AND, which uses the operator `&&` (single value) or `&` (vector)
* OR, which uses the operator `||` (single value) or `|` (vector)
  * This is the "pipe" character, usually located just above the backslash
* NOT, which uses the operator `!`


```r
> x = c(1, 2, 3)
> 
> # Use the vector operator '&' b/c testing each value in the vector (more common)
> x >= 1 & x < 3  
## [1]  TRUE  TRUE FALSE
> 
> # Use '&&' because only one value on each side
> sum(x) < 10 && mean(x) > 0 
## [1] TRUE
```
### Subsetting with boolean expressions
One of the most common things we do with Boolean operators is use them to subset our data


```r
> x = c(1, 2, 3, NA, 5)
> 
> # All values greater than 2
> x[x>2] 
## [1]  3 NA  5
> 
> # All values that are not missing
> x[!is.na(x)] 
## [1] 1 2 3 5
```
Subsetting data frames can be done either by using boolean indexes (like with vectors in the last example) or using the `subset()` function


```r
> berries = read.csv('data/blueberries.csv')
> 
> # Subset with boolean index
> berries[berries$variety=='Cerulean',] 
##    plot  variety resistance
## 10   10 Cerulean   4.969730
## 15   15 Cerulean   3.338443
## 21   21 Cerulean   1.590397
## 26   26 Cerulean   6.189447
## 30   30 Cerulean   3.729126
> 
> # Same, but with the subset() function
> subset(berries, berries$variety == 'Cerulean') 
##    plot  variety resistance
## 10   10 Cerulean   4.969730
## 15   15 Cerulean   3.338443
## 21   21 Cerulean   1.590397
## 26   26 Cerulean   6.189447
## 30   30 Cerulean   3.729126
```




## Other utility functions
### head() and tail()
head() and tail() show you the first/last few bits of a data structure. For vectors and lists, it shows the first/last items; for matrices and dataframes, they show the first/last rows.

```r
> df = data.frame(a=1:10, b=LETTERS[1:10])
> head(df) # First rows
##   a b
## 1 1 A
## 2 2 B
## 3 3 C
## 4 4 D
## 5 5 E
## 6 6 F
> tail(df) # Last rows
##     a b
## 5   5 E
## 6   6 F
## 7   7 G
## 8   8 H
## 9   9 I
## 10 10 J
> head(df$a) # First few elements of column a
## [1] 1 2 3 4 5 6
> head(df$a, n=8)# Specify how many you want to see
## [1] 1 2 3 4 5 6 7 8
```

### str()
str() (for "structure") will show you the structure of an object. This is extremely useful for finding out if your dataframe/list/whatever is actually put together correctly.

```r
> # Integer vector
> myvector = 1:20
> str(myvector) 
##  int [1:20] 1 2 3 4 5 6 7 8 9 10 ...
> 
> # Data frame; note it shows the data type of each column
> myframe = data.frame(a=1:10, b=LETTERS[1:10])
> str(myframe) 
## 'data.frame':	10 obs. of  2 variables:
##  $ a: int  1 2 3 4 5 6 7 8 9 10
##  $ b: Factor w/ 10 levels "A","B","C","D",..: 1 2 3 4 5 6 7 8 9 10
> 
> # List; also shows the type of each element and even nested sub-elements
> mylist = list(1:10, c("a","b","c"), mean, list(a=1:5, b=LETTERS[1:5]))
> str(mylist) 
## List of 4
##  $ : int [1:10] 1 2 3 4 5 6 7 8 9 10
##  $ : chr [1:3] "a" "b" "c"
##  $ :function (x, ...)  
##  $ :List of 2
##   ..$ a: int [1:5] 1 2 3 4 5
##   ..$ b: chr [1:5] "A" "B" "C" "D" ...
```

### paste()
paste() is what concatenates (joins) strings together. The *sep* argument determines what character(s) are put between the elements that are pasted

```r
> first = c("A","B","C")
> second = c(1,2,3)
> third = c("u","g","a")
> paste(first, second, sep="_")
## [1] "A_1" "B_2" "C_3"
```

paste() is especially useful if you have a data frame where plots have bene numbered "nested" inside fields and you need to make a unique label for each of them. (Similar situations come up when needing to uniquely label whole plots for split-plot designs, or strips for strip-plots.)

```r
> # Same data frame where plot numbers are nested within fields
> mydata = data.frame(field=c(1,1,2,2), plot=c(1,2,1,2))
> mydata
##   field plot
## 1     1    1
## 2     1    2
## 3     2    1
## 4     2    2
> 
> # First label fields and plots so are obviously factors
> mydata$field=paste("field", mydata$field, sep="")
> mydata$plot=paste("plot", mydata$plot, sep="")
> mydata
##    field  plot
## 1 field1 plot1
## 2 field1 plot2
## 3 field2 plot1
## 4 field2 plot2
> 
> # Now give each plot a unique ID by pasting the field and plot together
> mydata$plotID = paste(mydata$field, mydata$plot, sep="_")
> mydata
##    field  plot       plotID
## 1 field1 plot1 field1_plot1
## 2 field1 plot2 field1_plot2
## 3 field2 plot1 field2_plot1
## 4 field2 plot2 field2_plot2
```


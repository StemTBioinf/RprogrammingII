Foundations of R programming II
================
Shamit Soneji
12/10/2018

Shit gets real now.....
-----------------------

In the first R workshop you were shown the basics of the R language, subsetting vector and matricies, writing functions and basic plotting. Today we're going to roll all that together and start looking at how to wrap all these things up into an R package that will use S4 classes that you will make available on github.

This will be a fairly intense workshop, but if you can crack it will boost your knowledge and confidence.

The first thing we'll do is revisit `lists` from the first workshop. Here is the code we used to make one:

``` r
alpha <- LETTERS[1:8]
mat <- matrix(rnorm(40),nrow=8)
listex1  <- list(char=alpha,data=mat)
listex1
```

    ## $char
    ## [1] "A" "B" "C" "D" "E" "F" "G" "H"
    ## 
    ## $data
    ##            [,1]        [,2]       [,3]         [,4]        [,5]
    ## [1,] -0.5962447 -1.49025512  0.6501673 -0.008373624 -0.38890146
    ## [2,] -0.3321055 -1.34162257  0.5246728  0.810226439 -0.61983647
    ## [3,]  1.2874062 -1.74213240 -1.0998891  0.003657937  0.07420541
    ## [4,] -0.6575718 -0.01777613 -0.5374377  1.136077742 -0.36426138
    ## [5,] -3.0077702 -0.23055071  1.1144021 -0.782697230  0.66219859
    ## [6,]  0.5080752  0.29999909  0.2791978 -0.055043277  0.54565805
    ## [7,]  0.1310888 -0.67662655  0.5219338  0.825490016 -0.97519459
    ## [8,] -0.6932967  1.88066407  0.7954516 -0.829769965  0.41902319

You will rememver that elements of a list can be accessed using the `$` character, so:

``` r
listex1$data
```

    ##            [,1]        [,2]       [,3]         [,4]        [,5]
    ## [1,] -0.5962447 -1.49025512  0.6501673 -0.008373624 -0.38890146
    ## [2,] -0.3321055 -1.34162257  0.5246728  0.810226439 -0.61983647
    ## [3,]  1.2874062 -1.74213240 -1.0998891  0.003657937  0.07420541
    ## [4,] -0.6575718 -0.01777613 -0.5374377  1.136077742 -0.36426138
    ## [5,] -3.0077702 -0.23055071  1.1144021 -0.782697230  0.66219859
    ## [6,]  0.5080752  0.29999909  0.2791978 -0.055043277  0.54565805
    ## [7,]  0.1310888 -0.67662655  0.5219338  0.825490016 -0.97519459
    ## [8,] -0.6932967  1.88066407  0.7954516 -0.829769965  0.41902319

and subset in the usual way:

``` r
listex1$data[2:3,] #prints just rows 2 and 3
```

    ##            [,1]      [,2]       [,3]        [,4]        [,5]
    ## [1,] -0.3321055 -1.341623  0.5246728 0.810226439 -0.61983647
    ## [2,]  1.2874062 -1.742132 -1.0998891 0.003657937  0.07420541

Now we can write a function that takes the average of the columns in the matrix.

``` r
get.col.means <- function(lst){
  cl.mns <- apply(lst$data,2,mean)
  cl.mns
}
```

The funtion addresses the matrix in the list using `$data` and calculates the column means using apply as we did in in RI. No we can try this on our list:

``` r
get.col.means(listex1)
```

    ## [1] -0.42005234 -0.41478754  0.28106232  0.13744600 -0.08088858

So we can now calculate the column means of any matrix in a list where the matrix is addressed as `$nums`.

***Exercise*** Read in the single cell data we used in the last tutorial (<http://bone.bmc.lu.se/Public/Mouse_HSPC_reduced.txt>) and make a list called `hspc` where the expression values are placed in an element called `$data`.

``` r
exp.vals <- read.delim("Mouse_HSPC_reduced.txt",row.names=1,header=T,sep="")
hspc <- list(data=exp.vals)
```

Lets use our function from earlier to get the column means:

``` r
expr.col.mens <- get.col.means(hspc)
```

Oh fark! What happened?

The apply function expects a matrix. We can see what type of object hspc$data is by using:

``` r
class(hspc$data)
```

    ## [1] "data.frame"

Its a data.frame\`. You need to supply a matrix, so lets convert it:

``` r
hspc$data <- as.matrix(hspc$data)
expr.col.mens <- get.col.means(hspc)
expr.col.mens[1:10] # the first 10 avg values
```

    ##  LTHSC.1  LTHSC.2  LTHSC.3  LTHSC.4  LTHSC.5  LTHSC.6  LTHSC.7  LTHSC.8 
    ## 2.632397 2.762785 2.345211 2.230264 2.650563 3.210026 2.586950 3.188602 
    ##  LTHSC.9 LTHSC.10 
    ## 2.079043 2.682019

This is the problem with functions. They don't check what they are getting, so if they get something incompatible the code will fail and you'll get nothing.

This is why classes are a good idea. They are containers for data where the class/type of data needs to be stated up front so downstream functions get the correcty formatted objects.

### S4 classes

You can think of S4 classes as a list where everything is checked first before the list is made. Lets make a simple S4 class called `scell`:

``` r
setClass("scell",slots=c(data="matrix"))
```

This sets up an S4 class where it expects a matrix for it to be instantiated. Lets try and instantiate a `scell` class by calling `new` using the data you read into `exp.vals`:

``` r
hspc.s4 <- new("scell",data=exp.vals)
```

Didn't work did it? When we call new is cheks to see that the slot `data` is of type `matrix`. In this case we gave a `data.frame` which is why it failed. To make it work we need to do:

``` r
hspc.s4 <- new("scell",data=as.matrix(exp.vals))
```

This works. The `scell` object gets a required matrix and a new object `hspc.s4` which is an S4 class is made. Elemants of an S4 class are kept in `slots` and we can access them using the `@` symbol.

``` r
hspc.s4@data[1:10,1:10] # first 10 rows and 10 columns
```

    ##           LTHSC.1  LTHSC.2   LTHSC.3  LTHSC.4   LTHSC.5  LTHSC.6   LTHSC.7
    ## Kdm3a    0.000000 7.326561 1.1412773 0.000000 0.0000000 2.969495 0.0000000
    ## Coro2b   1.312809 5.699220 0.0000000 0.000000 0.0000000 6.115329 0.0000000
    ## Phf6     0.000000 1.685035 6.2547479 2.369840 2.1584379 7.823217 7.7356194
    ## Usp14    1.312809 3.301869 1.7704166 3.223063 6.1064430 8.544492 9.7926774
    ## Tmem167b 4.233503 0.000000 0.6806675 1.624999 0.0000000 2.969495 7.9009993
    ## Kbtbd7   0.000000 0.000000 0.6806675 0.000000 0.0000000 2.969495 0.0000000
    ## Rag2     1.312809 6.940771 1.1412773 5.102268 0.7594648 0.000000 0.0000000
    ## Hmgcs1   6.562359 1.685035 0.0000000 1.624999 9.4571134 2.969495 0.0000000
    ## Zfp947   1.988593 0.000000 0.0000000 0.000000 0.0000000 0.000000 0.0000000
    ## Atad2    8.444245 6.555359 1.7704166 2.858629 1.6222685 5.392170 0.7816531
    ##            LTHSC.8   LTHSC.9 LTHSC.10
    ## Kdm3a    3.9360487  0.000000 0.000000
    ## Coro2b   0.0000000  7.606749 0.000000
    ## Phf6     1.1110086  6.217745 3.919389
    ## Usp14    8.9796021  5.821215 1.835594
    ## Tmem167b 0.4716092  0.000000 0.000000
    ## Kbtbd7   2.6810387 10.910051 1.191937
    ## Rag2     0.0000000  0.000000 1.191937
    ## Hmgcs1   0.4716092  1.527437 3.122267
    ## Zfp947   0.4716092  0.000000 0.000000
    ## Atad2    8.4770954  2.512690 1.835594

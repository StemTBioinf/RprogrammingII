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
    ##             [,1]       [,2]        [,3]       [,4]        [,5]
    ## [1,]  0.87238280 -0.9412484  0.59460074  1.2691876  2.80806584
    ## [2,]  0.86642570 -0.6012126 -0.41001279 -0.4267889  1.43973427
    ## [3,] -0.30642222  1.2255967 -0.15296598 -1.3874687  0.23742448
    ## [4,] -0.26696880 -0.3354458 -0.01956228  0.7990826 -1.11846215
    ## [5,]  0.25451056  0.6429669 -1.38074950  1.6151788 -0.21512664
    ## [6,] -0.81673545 -0.2677959  0.10029964  0.4018044  0.04899058
    ## [7,]  0.06091362 -0.1014048  1.27198122 -0.3948766  0.84976128
    ## [8,]  1.26857438 -0.7411288 -0.54969860  0.1848064 -0.51534998

You will rememver that elements of a list can be accessed using the `$` character, so:

``` r
listex1$data
```

    ##             [,1]       [,2]        [,3]       [,4]        [,5]
    ## [1,]  0.87238280 -0.9412484  0.59460074  1.2691876  2.80806584
    ## [2,]  0.86642570 -0.6012126 -0.41001279 -0.4267889  1.43973427
    ## [3,] -0.30642222  1.2255967 -0.15296598 -1.3874687  0.23742448
    ## [4,] -0.26696880 -0.3354458 -0.01956228  0.7990826 -1.11846215
    ## [5,]  0.25451056  0.6429669 -1.38074950  1.6151788 -0.21512664
    ## [6,] -0.81673545 -0.2677959  0.10029964  0.4018044  0.04899058
    ## [7,]  0.06091362 -0.1014048  1.27198122 -0.3948766  0.84976128
    ## [8,]  1.26857438 -0.7411288 -0.54969860  0.1848064 -0.51534998

and subset in the usual way:

``` r
listex1$data[2:3,] #prints just rows 2 and 3
```

    ##            [,1]       [,2]       [,3]       [,4]      [,5]
    ## [1,]  0.8664257 -0.6012126 -0.4100128 -0.4267889 1.4397343
    ## [2,] -0.3064222  1.2255967 -0.1529660 -1.3874687 0.2374245

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

    ## [1]  0.24158507 -0.13995908 -0.06826344  0.25761570  0.44187971

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

***Exercise*** Write a function called `get.var.genes` that will take a `scell` S4 object and calculate the top N most variable genes (i.e we want a vector of gene names). The basis of the code is in the RI tutorial. Hint: the function that you create will need two input arguments.

``` r
get.var.genes <- function(sco,nvar){
  
  genes.var <- apply(sco@data,1,var)
  top.var.genes <- names(rev(sort(genes.var))[1:nvar])
  top.var.genes
}
```

Lets try and run this and get the top 10 maost variable genes:

``` r
get.var.genes(hspc.s4,10)
```

    ##  [1] "Elane" "Ctsg"  "Mpo"   "Car1"  "Ms4a3" "Mpl"   "Ly6c2" "Klf1" 
    ##  [9] "Nkg7"  "Ces2g"

Ok, this seems to work nicely. All we need to do now is put the results somewhere convenient, and the best place for this is back in the `hspc.s4` object in another slot called `var.genes`. Lets modify the function to do this:

``` r
get.var.genes <- function(sco,nvar){
  
  genes.var <- apply(sco@data,1,var)
  top.var.genes <- names(rev(sort(genes.var))[1:nvar])
  sco@var.genes <- top.var.genes #puts the var genes into a new slot 
  sco #returns the new object
}
```

Run it again:

``` r
hspc.s4 <- get.var.genes(hspc.s4,10)
```

Did it work? Did it f\*\*k. Why not?

We need to have a slot made in the S4 calss up front before we try to populate it. In this case we know that we have a characters being returned, so we it will be of type `character`.

``` r
setClass("scell",slots=c(data="matrix",var.genes="character"))
```

We need to make make a new instance of `hspc.s4` first so we have the new class definition:

``` r
hspc.s4 <- new("scell",data=as.matrix(exp.vals))
slotNames(hspc.s4) # we can see which slots you have available
```

    ## [1] "data"      "var.genes"

``` r
hspc.s4@var.genes #empty
```

    ## character(0)

Run the var genes function again:

``` r
hspc.s4 <- get.var.genes(hspc.s4,10)
hspc.s4@var.genes
```

    ##  [1] "Elane" "Ctsg"  "Mpo"   "Car1"  "Ms4a3" "Mpl"   "Ly6c2" "Klf1" 
    ##  [9] "Nkg7"  "Ces2g"

It works. This is the nice thing about S4 classes. You really need to think up-front what you need to store further down that line, and helps you regularise your functions. If we were using lists you could make slots on-the-fly and this normally leads to downstream chaos.

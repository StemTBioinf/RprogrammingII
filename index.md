Foundations of R programming II
================
Shamit Soneji & Stefan Lang

The fun starts now...
---------------------

In the first R workshop you were shown the basics of the R language, subsetting vector and matrices, writing functions and basic plotting. Today we're going to roll all that together and start looking at how to wrap all these things up into an R package that will use S4 classes that you will make available on github.

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
    ##            [,1]       [,2]       [,3]       [,4]        [,5]
    ## [1,] -0.7676074  0.2040270 -0.8058903 -0.1121556  1.04619760
    ## [2,] -1.0519096  1.5053127  1.3976931  0.8084768 -0.11235048
    ## [3,] -1.1007545 -0.4551513 -0.2645622  0.3948332  0.02805027
    ## [4,]  0.7186680 -0.9424725  0.7567774  0.7667434  0.85084991
    ## [5,]  0.4646616 -1.1350542  0.2644048  0.4315982 -1.40096608
    ## [6,] -0.6140013  0.5672753 -1.8712543 -0.5663047 -0.01961996
    ## [7,]  1.3777421 -0.9193192  0.8301587  1.2489152 -0.76952136
    ## [8,]  0.2166990 -0.1375273 -0.4085618  0.6884870 -0.53285209

You will rememver that elements of a list can be accessed using the `$` character, so:

``` r
listex1$data
```

    ##            [,1]       [,2]       [,3]       [,4]        [,5]
    ## [1,] -0.7676074  0.2040270 -0.8058903 -0.1121556  1.04619760
    ## [2,] -1.0519096  1.5053127  1.3976931  0.8084768 -0.11235048
    ## [3,] -1.1007545 -0.4551513 -0.2645622  0.3948332  0.02805027
    ## [4,]  0.7186680 -0.9424725  0.7567774  0.7667434  0.85084991
    ## [5,]  0.4646616 -1.1350542  0.2644048  0.4315982 -1.40096608
    ## [6,] -0.6140013  0.5672753 -1.8712543 -0.5663047 -0.01961996
    ## [7,]  1.3777421 -0.9193192  0.8301587  1.2489152 -0.76952136
    ## [8,]  0.2166990 -0.1375273 -0.4085618  0.6884870 -0.53285209

and subset in the usual way:

``` r
listex1$data[2:3,] #prints just rows 2 and 3
```

    ##           [,1]       [,2]       [,3]      [,4]        [,5]
    ## [1,] -1.051910  1.5053127  1.3976931 0.8084768 -0.11235048
    ## [2,] -1.100754 -0.4551513 -0.2645622 0.3948332  0.02805027

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

    ## [1] -0.09456276 -0.16411369 -0.01265433  0.45757419 -0.11377652

So we can now calculate the column means of any matrix in a list where the matrix is addressed as `$nums`.

***Exercise*** Read in the single cell data we used in the last tutorial (<http://bone.bmc.lu.se/Public/Mouse_HSPC_reduced_v2.txt>) and make a list called `hspc` where the expression values are placed in an element called `$data`. This is from the same dataset, but slightly different, hence "v2" in the name.

``` r
exp.vals <- read.delim("Mouse_HSPC_reduced_v2.txt",row.names=1,header=T,sep="\t")
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
    ## 2.424127 2.242731 2.629759 2.356785 2.706467 2.966355 3.026223 2.662917 
    ##  LTHSC.9 LTHSC.10 
    ## 2.915014 3.050313

This is the problem with functions that are applied to lists etc. They do not check what they are getting, so if they get something incompatible the code will fail and you'll get nothing back.

This is why classes are a good idea. They are containers for data where the class/type of data needs to be stated up front so downstream functions get the correctly formatted objects.

### S4 classes

You can think of S4 classes as a list where everything is checked first before the S$ class is made. Lets make a simple S4 class called `scell`:

``` r
setClass("scell",slots=c(data="matrix"))
```

This sets up an S4 class where it expects a matrix for it to be instantiated. Lets try and instantiate a `scell` class by calling `new` using the data you read into `exp.vals`:

``` r
hspc.s4 <- new("scell",data=exp.vals)
```

Didn't work did it? When we call new is checks to see that the slot `data` is of type `matrix`. In this case we gave a `data.frame` which is why it failed. To make it work we need to do:

``` r
hspc.s4 <- new("scell",data=as.matrix(exp.vals))
```

This works. The `scell` object gets a required matrix and a new object `hspc.s4` which is an S4 class is made. Elements of an S4 class are kept in `slots` and we can access them using the `@` symbol.

``` r
hspc.s4@data[1:10,1:10] # first 10 rows and 10 columns
```

    ##           LTHSC.1  LTHSC.2  LTHSC.3   LTHSC.4   LTHSC.5  LTHSC.6  LTHSC.7
    ## Kdm3a    0.000000 1.079653 0.000000 1.1412773 1.7726083 2.015252 1.981955
    ## Coro2b   0.000000 1.079653 0.000000 0.0000000 0.0000000 0.000000 0.000000
    ## Phf6     7.271829 2.117889 8.706240 6.2547479 8.4504358 2.824772 6.530737
    ## Usp14    3.378561 3.136633 6.286004 1.7704166 7.4176445 9.244268 8.293378
    ## Tmem167b 9.093340 2.117889 2.170399 0.6806675 0.8524546 2.824772 0.000000
    ## Kbtbd7   2.861341 9.705688 0.000000 0.6806675 1.3846802 2.015252 0.000000
    ## Rag2     2.047347 0.000000 1.459819 1.1412773 9.3556223 0.000000 4.704727
    ## Hmgcs1   8.191854 2.117889 0.000000 0.0000000 6.4328001 0.000000 3.404414
    ## Zfp947   0.000000 0.000000 0.000000 0.0000000 0.0000000 0.000000 4.822225
    ## Atad2    2.861341 2.715375 3.000533 1.7704166 7.2521661 3.340211 5.377344
    ##            LTHSC.8   LTHSC.9 LTHSC.10
    ## Kdm3a    0.0000000 0.8156131 1.671105
    ## Coro2b   0.0000000 0.0000000 0.000000
    ## Phf6     2.1584379 4.2661196 6.840443
    ## Usp14    6.1064430 3.2265864 4.645611
    ## Tmem167b 0.0000000 1.7137564 2.917194
    ## Kbtbd7   0.0000000 0.8156131 2.424699
    ## Rag2     0.7594648 1.3334763 0.000000
    ## Hmgcs1   9.4571134 2.6599980 3.283676
    ## Zfp947   0.0000000 2.2631034 1.671105
    ## Atad2    1.6222685 1.3334763 4.513860

***Exercise*** This is fine, but by calling `new` the user still has to remember that the data has to be of class `matrix`. Think of a function that could be written to make the life of a user easier. Hint: Have a look at `is.matrix` in the help section.

***Exercise*** Write a function called `get.var.genes` that will take a `scell` S4 object and calculate the top N most variable genes (i.e we want a vector of gene names). The basis of the code is in the RI tutorial.

Hint: the function that you create will need two input arguments.

``` r
get.var.genes <- function(sco,nvar){
  
  genes.var <- apply(sco@data,1,var)
  top.var.genes <- names(rev(sort(genes.var))[1:nvar])
  top.var.genes
}
```

Lets try and run this and get the top 10 moast variable genes:

``` r
get.var.genes(hspc.s4,10)
```

    ##  [1] "Elane"     "Ctsg"      "Mpo"       "Car1"      "Ms4a3"    
    ##  [6] "Ly6c2"     "Klf1"      "Prtn3"     "Serpina3g" "Aqp1"

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

We need to have a slot made in the S4 class up front before we try to populate it. In this case we know that we have a characters being returned, so we it will be of type `character`.

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

    ##  [1] "Elane"     "Ctsg"      "Mpo"       "Car1"      "Ms4a3"    
    ##  [6] "Ly6c2"     "Klf1"      "Prtn3"     "Serpina3g" "Aqp1"

It works. This is the nice thing about S4 classes. You really need to think up-front what you need to store further down that line, and helps you regularise your functions. If we were using lists you could make slots on-the-fly and this normally leads to downstream chaos.

### Packages

After a while you start to develop a large back of functions and classes which you use routinely in your work as you get more confident working in R. While you could keep all these functions in an R script, the nicer thing to do would be to roll them all into a package. Packages also make you do something very important, and that is document your code. Useful for when you go back to things after a long time. To do this we need to install two packages to help us, `devtools` and `roxygen2`.

``` r
install.packages("devtools")
install.packages("roxygen2")
```

Then call the libraries:

``` r
library(devtools)
library(roxygen2)
```

Make a new folder called `RPackages` on your laptop and change you working session to it.

You create the start of a new package using the `create` function. Create a package called `MyFirstPackage`:

``` r
create_package("MyFirstPackage")
## use the find command to inspect the new folder structure
system( 'find ./')
```

This function will create an initial package folder and directly change the working directory to the new folder. If you go the files panel on the bottom-right side you will see a "Files" tab. Use that to navigate to the "MyFirstPackage" folder. In there you will see a few things. Open the "DESCRIPTION" file to see what's in it.

Lets start banking our functions in this package. Go to "File" -&gt; "New File" -&gt; "R Script"

If you design you package as many small R scripts that define exactly one function and are also named like the function the development of this package will become a lot easier.

In this script the only thing we want to do is define the class - plase name the script '01.class.R'. With 01 at the beginning it will always sort first in your directory listings.

The file content looks a little like this:

``` r
#'Class defintion of an scell object
#`
#`The class takes a matrix of values and needs row and column names.
setClass("scell",slots=c(data="matrix",var.genes="character"))
```

Thats it! Save the script in the "R" folder which is where all the scripts should be kept. Call it `scellClass.R`. What you need to do now is call the `document()` function:

``` r
document()
```

This takes the lines of documentation that you created an forms the manual that you have in the newly created "man" folder. It also updates other files automatically such as the NAMESPACE file.

### The first function get.var.genes

Now lets make a new R script that contains the function that gets the variable genes. Open a new R scripts and put in the following:

``` r
#'Calculates the top N variable genes from a scell object
#'@param sco An scell object
#'@param nvar The number of genes we wich to retrieve
#'@export get.var.genes
get.var.genes <- function(sco,nvar){
  
  genes.var <- apply(sco@data,1,var)
  top.var.genes <- names(rev(sort(genes.var))[1:nvar])
  sco@var.genes <- top.var.genes #puts the var genes into a new slot 
  sco #returns the new object
}
```

Save the file as "GetVarGenes.R" in the "R" folder and now run:

``` r
document()
```

The new function will be added to the manual pages, and thats it, you have now made your first R package that lets you make a new scell class and then calculate the N top variable genes. All we need to do now is install it, and we do this by issuing the command:

``` r
install()
```

Done! Whenever we fire up R we can now call our package using:

``` r
library(MyFirstPackage)
```

Lets try it out:

``` r
exp.vals <- read.delim("Mouse_HSPC_reduced.txt",row.names=1,header=T,sep="")
hspc.s4 <- new("scell",data=as.matrix(exp.vals))
hspc.s4 <- get.var.genes(hspc.s4,500)
hspc.s4@var.genes[1:10]
```

### Your first own functions

We have the rudiments of a package here, so lets expand on it some more and make it do two more things:

-   Make a function that reads the input file and makes an scell object immediately.
-   Plots a heatmap of the N variable genes you find from using the earlier function.

Go!

One more thing to help you: You can use the devtools::check() function to highlight problems in your code/functions.

### Define dependency packages

Now we're going to extend the capabilities of our package by making a function that will perform a tSNE of the data and plot it in 3D. The first thing we need to do is make sure `MyFirstPackage` can also install other packages it needs to function. In this case we need the `Rtsne` and the `rgl` package. To do this open up the "DESCRIPTION" file and alter it to look like this:

``` r
Package: MyFirstPackage
Title: What the Package Does (one line, title case)
Version: 0.0.0.9000
Authors@R: person("First", "Last", email = "first.last@example.com", role = c("aut", "cre"))
Description: What the package does (one paragraph).
Suggests: 
  testthat
Depends: R (>= 3.4.0),
  methods,
  stats,
  utils,
  pheatmap,
  Rtsne,
  rgl
License: GLP-3
Encoding: UTF-8
LazyData: true
RoxygenNote: 6.0.1
```

What happens now is that when `MyFirstPackage` is installed `Rtsne` and `rgl` packages will be installed if they aren't already. Issue the `document()` and `install()` command again after you have saved this file:

``` r
install()
```

You will see that it now installs the required extra packages.

### Implement a CalcTSNE function

***Exercise*** Go the help page for `Rtsne` and work out how to use it. Use it to calculate a tSNE over 3 dimensions on the hspc.s4 data using the variable genes only. Find where these coordinates are kept in the output.

``` r
tsne.out <- Rtsne(hspc.s4@data[hspc.s4@var.genes,]),dims = 3)
dim(tsne.out)
class(tsne.out)
```

***Exercise*** Write a function called `CalcTSNE` that takes a `scell` object and a variable `ndim` (that indicates how may dimensions you want to calculate over) and calculates the tSNE and then puts the coordinates in a slot called `tsne`. Put this into your package in a script called `DimensionReduction.R` and reinstall your package.

Lets try it out on our `hspc.s4` object:

``` r
library(MyFirstPackage)
hspc.s4 <- CalcTSNE(hspc.s4,3)
hspc.s4@tsne
```

We can plot these using the `rgl.points` function:

``` r
library(rgl)
rgl.points(hspc.s4@tsne)
```

At this point you now have the rudiments of an R package that you can expand on further. You don't have to build them around S4 classes, but it helps.

Getting your package out there with Github
------------------------------------------

This is all well and good, but what do you do if you need your R package and you don't have your laptop, or more often, you want to share you package/code with a collaborator? This is where Github (<https://github.com/>) comes in really handy. Github is an online repositor for code popular with most developers.

Before this workshop you made an account for Github so login now and do the following:

1.  Create a new repository and call it `MyFirstPackage`
2.  When you have done this you will see some instructions on getting your file up.

***Get your files up:***

In the path of your package do this:

    echo "# MyFirstPackage" >> README.md
    git init
    git add --all .
    git commit -m "first commit"
    git remote add origin YourPackageGitRepoCoordinates
    git push -u origin master

The easiest way to install this package is from within an R console:

``` r
## e.g. the example I created last week:
devtools::install_git('https://github.com/StemTBioinf/Example_MyFirstPackage.git')
```

Now that you know how to make a package and push it to GitHub we can now go full hacker-mode and expand the features of our `scell` class and funtions even more. Lets do a few more things to make the `scell` class more useful:

1.  Here is a file if index sorting data (<http://bone.bmc.lu.se/Public/Mouse_HSPC_reduced_IndexSortData.txt>). These cells were assayed for single-cell expression in the dataset we are using. Expand your package so that index sorting data can be loaded into a `facs` slot in your `scell` class.

2.  Write some code that allows you to cluster and split the data into N partitions and store the cluster memberships somewhere in your class.

3.  Write a function that for any specified cluster the finction will produce a violin or beanplot for each of the markers in the facs data to show the overall surface marker profiles of the selected cluster.

4.  Write a function that will produce a 3D tSNE plot where each cell is coloured according to the cluster it belongs to, i.e 5 clusters means 5 colours in the plot.

5.  Write a function that for a given gene will plot a 3D tSNE where each cell is coloured according to the intensity of the genes to give an idea of the gene's expression levels over the dataset.

***BONUS exercise:*** For any given cluster, find a package that will allow you to workout over-represented gene ontologies (GO) from the genes in a specified cluster. This might be a pain, lets see how far we get.

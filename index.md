Foundations of R programming II
================
Shamit Soneji
12/10/2018

Shit gets real now.....
-----------------------

In the first R workshop you were shown the basics of the R language, subsetting vector and matricies, writing functions and basic plotting. Today we're going to roll all that together and start looking at how to wrap all these things up into an R package that will use S4 classes that you will make available on github.

This will be a fairly intense workshop, but if you can crack it will boost your knowledge and confidence.

The first thing we'll do is revisit `lists` from the first workshop. Here is the code we used to make one:
----------------------------------------------------------------------------------------------------------

``` r
alpha <- LETTERS[1:8]
mat <- matrix(rnorm(40),nrow=8)
listex1  <- list(char=alpha,nums=mat)
listex1
```

    ## $char
    ## [1] "A" "B" "C" "D" "E" "F" "G" "H"
    ## 
    ## $nums
    ##            [,1]       [,2]       [,3]        [,4]        [,5]
    ## [1,]  2.2582412  1.3951656 -0.3716412  0.43447992 -0.67780095
    ## [2,]  0.8531906  2.3135577 -0.5638934 -0.36232839  0.04679281
    ## [3,]  2.0093960  1.5198798  1.4133207 -0.25596139  1.43114618
    ## [4,] -1.4071409  0.4383417  0.2570399  0.88159322  0.40895242
    ## [5,]  0.4355281  2.0897386 -0.4906181  0.05150121  0.87246356
    ## [6,]  3.3315012 -0.7369497  0.2212340 -0.63476550 -1.01099754
    ## [7,] -0.7866181  1.8163536 -2.2027455 -1.49987632  0.39051573
    ## [8,] -1.6679528  0.1542561  1.5297016 -0.15944683 -0.10736824

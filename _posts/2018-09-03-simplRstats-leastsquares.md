Least Squares model fitting
---------------------------

Generate some data, removing anything too close to zero for clarity

``` r
distX <- seq(-200, 200,5)
distX <- distX[abs(distX)>30] # remove any values close to the  
dist <- rep(distX,each=10)
```

Generate the response variables with some noise

``` r
genresponse<- function(distances, type){
  if(type=="flat"){
    resp <- ifelse(distances >0, 
                  rnorm(length(distances), (4 + 10*exp(-0.05 * distances)), 0.2), 
                  rnorm(length(distances), 4,0.2))
  }
  if(type=="proximity"){
    #' Could expand model to account for higher values closer to the zero 
    resp <- ifelse(distances >0, 
                  rnorm(length(distances), (4 + 10*exp(-0.05 * distances)), 0.2), 
                  rnorm(length(distances), (4+2*exp(0.1*distances)),0.2))
  }
  resp
}
resX <- genresponse(distX, type="flat")
res <- genresponse(dist, type="flat")
```

Plot the data

``` r
plot(distX, resX)
```

![](least_squares_files/figure-markdown_github/unnamed-chunk-3-1.png)

``` r
plot(dist, res)
```

![](least_squares_files/figure-markdown_github/unnamed-chunk-3-2.png)

Write a model for a linear relationship where x is negative and an exponential where y is positive

``` r
mymodel <- function(par, d){
  # par= three model parameters
  # d =distance to edge 
  model <- ifelse(d<0,
                  par[1],
                  par[1] + par[2]*exp(-par[3]*d))
}
```

Predict from this model to check it works as expected

``` r
pred <- mymodel(par=c(4,8, 0.03), d=dist)
plot(dist, res, col="grey40"); lines(dist, pred, col="blue")
```

![](least_squares_files/figure-markdown_github/unnamed-chunk-5-1.png) Not great but it will do as a starting point

Fit the model using least squares

``` r
lsq <- function(par, spmd, d){
  model <- ifelse(d<0, par[1],par[1] + par[2]*exp(-par[3]*d))
  rss <- sum((spmd-model)^2)
  rss
}

params <- optim(c(4, 8, 0.04),lsq, spmd=res, 
               d=dist)
params$par
```

    ## [1]  3.99346239 10.60460557  0.05134586

Predict from the model using the optimised parameters and plot the final results

``` r
pred2 <- mymodel(par=params$par, d=dist)
plot(dist, res, col="grey40"); lines(dist, pred2, col="blue", lwd=2)
```

![](least_squares_files/figure-markdown_github/unnamed-chunk-7-1.png)

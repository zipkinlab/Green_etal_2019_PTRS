Vignette 1
================

##### Model for vignette 1

``` r
model{

#--------#
#-Priors-#
#--------#

#Intercept
beta0 ~ dnorm(0, 0.1)
#Autoregressive effect
beta1 ~ dnorm(0, 0.1)
#Lag effect of subgroup size @ kills
for(k in 1:nclan){
beta2[k] ~ dnorm(0, 0.1)
}#end k loop
#Random effect of year (time)
for(t in 1:nyrs){
beta3[t] ~ dnorm(0, tau.t)
}#end t loop
tau.t ~ dgamma(0.1, 0.1) #Precision for RE of year
tau.p <- 1/(sig.p*sig.p) #Precision for process variation
sig.p ~ dunif(15, 25) #Process variation
#First year mean clan size
mu[21,1] ~ dunif(15, 35) #Happy Zebra
mu[21,2] ~ dunif(30, 50) #Serena North
mu[21,3] ~ dunif(30, 50) #Serena South
mu[1,4] ~ dunif(50, 70) #Talek West

#------------#
#-Likelihood-#
#------------#

for(i in 1:nobs){
#Normally distributed clan size
clan.size[i] ~ dnorm(mu[yr[i], clan[i]], tau.p)
#Format group size, i, by t,k
gs[yr[i], clan[i]] <- group[i]
}#end i loop
for(k in 1:nclan){
for(t in ns[k]:ne[k]){
#Linear predictor of mean clan size by clan
mu[t,k] <- beta0 +                    #Intercept
               beta1 * mu[t-1,k] +    #Autoregressive effect
               beta2[k] * gs[t-1,k] + #Lag effect of subgroup size @ kills
               beta3[t]               #Random effect of year (time)
}#end t loop
}#end k loop

}
```

``` r
#Evaluate this R chunk if Vignette1.Rdata has not been produced.

#data$DSGclan.size[is.na(data$DSGclan.size)] <- data$clan.size[is.na(data$DSGclan.size)]

#-Bugs Data-#
bugs.data <- list(clan.size = data$DSGclan.size, clan = as.numeric(data$clan), 
                  yr = data$year - 1989 + 1, nyrs = max(data$year) - 1989 + 1, nobs = 54,
                  ns = ns, ne = ne, nclan = 4, group = as.numeric(scale(data$gs.kill)))

#-Parameters-#
params <- c("beta0", "beta1", "beta2", "sig.p", "tau.t")

#-MCMC settings-#
nb <- 91000
ni <- 100000
nt <- 3
nc <- 3
na <- 100

#-Inits-#
inits <- function(){list(beta0 = runif(1, -1.5, -1), beta1 = runif(1, 1, 1.1))}

#-Run clan size model-#
out <- jagsUI(bugs.data, inits, params, "Vignette1.txt", n.thin=nt, 
               n.chains=nc, n.burnin=nb, n.iter=ni, n.adapt=na,  parallel = TRUE)

save(out, file = "Vignette1.Rdata")
```

``` r
#Evaluate this R chunk if Vignette1.Rdata already exist.
load(file = "Vignette1.Rdata")
```

Vignette 1 Table S1. JAGS output for the above model.

|            |    mean|    2.5%|   97.5%|     f|
|:-----------|-------:|-------:|-------:|-----:|
| beta0      |    0.53|   -3.51|    4.63|  0.60|
| beta1      |    1.03|    0.96|    1.09|  1.00|
| beta2\[1\] |   -1.14|   -5.95|    3.64|  0.68|
| beta2\[2\] |    0.20|   -4.46|    4.85|  0.53|
| beta2\[3\] |   -0.36|   -5.68|    5.00|  0.55|
| beta2\[4\] |    4.73|    0.55|    8.25|  0.99|
| sig.p      |   15.46|   15.01|   16.68|  1.00|
| tau.t      |    1.39|    0.02|   11.30|  1.00|
| deviance   |  415.77|  407.17|  426.68|  1.00|

![](Vignette1_files/figure-markdown_github/unnamed-chunk-7-1.png)

Figure 3. Annual mean estimates of the total size of each of the four hyena clans monitored. One year lag effect of mean hyena subgroup size found at kills on clan size for each of the four clans. We estimated a 99.8% probability that subgroup size had a positive lag effect on clan size in Talek, but no effect was estimated on the other clans.

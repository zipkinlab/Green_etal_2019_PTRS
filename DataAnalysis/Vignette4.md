Vignette 4
================

Please see [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1413563.svg)](https://doi.org/10.5281/zenodo.1413563) or <https://github.com/farrmt/HMSDS> for code on multi-species distance sampling.

##### Model for single-species framework for black-backed jackal and spotted hyenas.

``` r
model{

#--------#
#-PRIORS-#
#--------#

#Gamma
r.N ~ dgamma(0.1, 0.1) #Number of subgroups
r.G ~ dunif(0,20)      #Subgroup size

#Expected Group Size
beta0 ~ dnorm(0, 0.1)          #intercept parameter
beta1 ~ dnorm(0, 0.1)
beta2 ~ dnorm(0, 0.1)
beta3 ~ dnorm(0, 0.1)
beta4 ~ dnorm(0, 0.1)

#Psi
tau_p ~ dgamma(0.1, 0.1)  #Precision
sig_p <- 1/sqrt(tau_p) #Variance

#Detection
gamma0 ~ dnorm(0, 0.1)
gamma1 ~ dnorm(0, 0.1)

#Expected Number of Groups
alpha0 ~ dnorm(0, 0.1)    #intercept parameter
alpha1 ~ dnorm(0, 0.1)
alpha2 ~ dnorm(0, 0.1)
alpha3 ~ dnorm(0, 0.1)
alpha4 ~ dnorm(0, 0.1)

for(j in 1:nsites){

sigma[j] <- exp(gamma0 + gamma1 * region[j])

psi[j] ~ dnorm(0, tau_p)       #transect effect parameter

#------------#
#-LIKELIHOOD-#
#------------#

for(t in 1:nreps[j]){

#Construct cell probabilities for nG cells using numerical integration
#Sum of the area (rectangles) under the detection function

for(k in 1:nD){

#Half normal detection function at midpt (length of rectangle)
g[k,t,j] <- exp(-mdpt[k]*mdpt[k]/(2*sigma[j]*sigma[j]))

#Proportion of each interval (width of rectangle) for both sides of the transect
pi[k,t,j] <- v/B

#Detection probability for each distance class k (area of each rectangle)
f[k,t,j] <- g[k,t,j] * pi[k,t,j]

#Conditional detection probability (scale to 1)
fc[k,t,j] <- f[k,t,j]/pcap[t,j]

}#end k loop

#Detection probability at each transect (sum of rectangles)
pcap[t,j] <- sum(f[1:nD,t,j])

#Observed population @ each t,j (N-mixture)
y[t,j] ~ dbin(pcap[t,j], N[t,j])

#Latent Number of Groups @ each t,j (negative binomial)
N[t,j] ~ dpois(lambda.star[t,j])

#Expected Number of Groups
lambda.star[t,j] <- rho[t,j] * lambda[t,j]

#Overdispersion parameter for Expected Number of Groups
rho[t,j] ~ dgamma(r.N, r.N)

#Linear predictor for Expected Number of Groups
lambda[t,j] <- exp(alpha0 + alpha1 * Disturbance[j] + alpha2 * Cattle[t,j] + alpha3 * Shoat[t,j] + alpha4 * Lions[t,j] + psi[j] + log(offset[j]))

#Expected Group Size
gs.lam.star[t,j] <- gs.lam[t,j] * gs.rho[t,j]

#Overdispersion parameter for Expected Group Size
gs.rho[t,j] ~ dgamma(r.G, r.G)

#Linear predictor for Expected Group Size
gs.lam[t,j] <- exp(beta0 + beta1 * Disturbance[j] + beta2 * Cattle[t,j] + beta3 * Shoat[t,j] + beta4 * Lions[t,j] + log(offset[j])) 

#Abundance per transect
GSrep[t,j] <- lambda.star[t,j] * gs.lam.star[t,j]

}#end t loop

#Abundance per transect averaged over surveys
GSsite[j] <- mean(GSrep[1:nreps[j], j])

#Mean detection probability @ each j
psite[j] <- mean(pcap[1:nreps[j], j])

}#end j loop

#Mean detection probability for each species
Dprop <- mean(psite[1:nsites])

#Mean abundance per transect
GS <- mean(GSsite[1:nsites])   

#Abundance per transect for each region
RegGS[1] <- mean(GSsite[1:13])     #Mara Triangle
RegGS[2] <- mean(GSsite[14:17])    #Talek region

for(i in 1:nobs){

#Observed distance classes
dclass[i] ~ dcat(fc[1:nD, rep[i], site[i]])

#Observed Group Size (zero truncated negative binomial)
gs[i] ~ dpois(gs.lam.star[rep[i], site[i]]) T(1,)

}#end i loop

}
```

![](../Figures/Figure6.tiff)

Figure 6. Parameter estimates from a community model (a,b) and a single-species model for black-backed jackal and spotted hyena (c,d) showing the effects of disturbance (log scale) on the number of groups and subgroup sizes. Note that a positive effect of disturbance is interpreted as a higher value in the Talek region than in the Mara Triangle. Mean values are indicated with small horizontal bars; 50% and 95% credible intervals are shown with thick and thin vertical bars, respectively. (a) Effect of disturbance on the expected number of individuals or subgroups for all species as estimated using a community model. (b) Effect of disturbance on the expected subgroup size for each species estimated with a community model. (c) Estimates of the effects of disturbance, African lion, cattle and sheep/goats on the expected number of black-backed jackal (BBJ) and spotted hyena (SH) subgroups estimated in single-species models. (d) Estimates of the effects of disturbance, African lion, cattle and sheep/goats on the expected subgroup size of black-backed jackal and spotted hyena estimated with singlespecies models.

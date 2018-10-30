# Vignette #1 Code

Model for vignette #1. 

```r
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


Vignette 1 Table S1. JAGS output for the above model.
<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> mean </th>
   <th style="text-align:right;"> 2.5% </th>
   <th style="text-align:right;"> 97.5% </th>
   <th style="text-align:right;"> f </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> beta0 </td>
   <td style="text-align:right;"> 1.25 </td>
   <td style="text-align:right;"> -2.30 </td>
   <td style="text-align:right;"> 5.48 </td>
   <td style="text-align:right;"> 0.72 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beta1 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 0.93 </td>
   <td style="text-align:right;"> 1.06 </td>
   <td style="text-align:right;"> 1.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beta2[1] </td>
   <td style="text-align:right;"> -0.43 </td>
   <td style="text-align:right;"> -5.27 </td>
   <td style="text-align:right;"> 4.45 </td>
   <td style="text-align:right;"> 0.57 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beta2[2] </td>
   <td style="text-align:right;"> 1.06 </td>
   <td style="text-align:right;"> -3.50 </td>
   <td style="text-align:right;"> 5.61 </td>
   <td style="text-align:right;"> 0.68 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beta2[3] </td>
   <td style="text-align:right;"> 0.19 </td>
   <td style="text-align:right;"> -5.10 </td>
   <td style="text-align:right;"> 5.52 </td>
   <td style="text-align:right;"> 0.53 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> beta2[4] </td>
   <td style="text-align:right;"> 4.98 </td>
   <td style="text-align:right;"> 1.95 </td>
   <td style="text-align:right;"> 7.91 </td>
   <td style="text-align:right;"> 1.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sig.p </td>
   <td style="text-align:right;"> 15.44 </td>
   <td style="text-align:right;"> 15.01 </td>
   <td style="text-align:right;"> 16.65 </td>
   <td style="text-align:right;"> 1.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> tau.t </td>
   <td style="text-align:right;"> 3.27 </td>
   <td style="text-align:right;"> 0.06 </td>
   <td style="text-align:right;"> 19.25 </td>
   <td style="text-align:right;"> 1.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deviance </td>
   <td style="text-align:right;"> 413.34 </td>
   <td style="text-align:right;"> 406.77 </td>
   <td style="text-align:right;"> 423.42 </td>
   <td style="text-align:right;"> 1.00 </td>
  </tr>
</tbody>
</table>
![](https://raw.githubusercontent.com/farrmt/ZSL/master/Figures/Figure3.png)

Figure 3. One year lag effect of mean group size @ kills per year on clan size. There is a 99.8% probability that group size has a positive lag effect on clan size.

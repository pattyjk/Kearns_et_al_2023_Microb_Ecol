## Comparing taxonomy assignments between databases

```
#read in data
unite<-read.delim("unite_taxonomy.txt", header=T)
warcup<-read.delim("warcup_taxonomy.txt", header=T)
ncbi<-read.delim("ncbi_taxonomy.txt", header=T)

#melt data for plotting
library(reshape2)
unite_m<-melt(unite)
names(unite_m)<-c("Tax", 'SampleID', "Rel_abun_unite")
warcup_m<-melt(warcup)
names(warcup_m)<-c("Tax", 'SampleID', "Rel_abun_warcup")
ncbi_m<-melt(ncbi)
names(ncbi_m)<-c("Tax", 'SampleID', "Rel_abun_ncbi")

#make plot
par(mfrow=c(1,3))
plot(unite_m$Rel_abun_unite, warcup_m$Rel_abun_warcup, xlab='Relative Abundance- Unite', ylab='Relative Abundance- Warcup', main='(A)')
abline(0,1, col='red')
plot(unite_m$Rel_abun_unite, ncbi_m$Rel_abun_ncbi, xlab='Relative Abundance- Unite', ylab='Relative Abundance- NCBI', main='(B)')
abline(0,1, col='red')
plot(ncbi_m$Rel_abun_ncbi, warcup_m$Rel_abun_warcup, xlab='Relative Abundance- NCBI', ylab='Relative Abundance- Warcup', main='(C)')
abline(0,1, col='red')

#spreaman correlation
cor.test(unite_m$Rel_abun_unite, warcup_m$Rel_abun_warcup, method='spearman')
#rho = 0.85, p<2e-16
cor.test(unite_m$Rel_abun_unite, ncbi_m$Rel_abun_ncbi, method='spearman')
#rho =.85, p<2e-16
cor.test(ncbi_m$Rel_abun_ncbi, warcup_m$Rel_abun_warcup, method='spearman')
#rho =.85,p<2e-16

```

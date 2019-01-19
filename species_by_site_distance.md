## Community similarity between species within/between sites
```
library(vegan)
library(ggplot2)
library(reshape2)
library(plyr)

#read in OTU table
otu_table<-read.delim("bat_mycobiome/otu_table_tax_R.txt", row.names=1, header=T)
otu_table<-otu_table[,-165]
otu_table_t<-t(otu_table)
#otu_table_t<-rrarefy(otu_table_t, sample=1200)

#calculate Bray-Curtis similarity
com_dist<-as.data.frame(as.matrix(vegdist(otu_table_t, method='horn')))
com_dist$sample1<-names(com_dist)

com_dist_m<-melt(com_dist)
names(com_dist_m)<-c("sample1", 'sample2', 'bc_dis')

#read in meta data
meta<-read.delim("bat_mycobiome/uparse_mapping_file.txt", header=T)

#create new data frame just with 'SampleID', 'Species', and 'area'
site_samp<-meta[,c('SampleID', 'area', 'species')]

#add site information to Bray-Curtis similarity values
com_dist_m<-merge(com_dist_m, site_samp, by.x = 'sample1', by.y='SampleID')
names(com_dist_m)<-c("sample1", 'sample2', 'bc_sim', 'site1', 'species1')

com_dist_m<-merge(com_dist_m, site_samp, by.x = 'sample2', by.y='SampleID')
names(com_dist_m)<-c("sample1", 'sample2', 'bc_sim', 'site1', 'species1', 'site2', 'species2')
dist_same
dim(com_dist_m)
#26569 by 7

#same site
dist_same<-com_dist_m[com_dist_m$site1 == com_dist_m$site2,]
dim(dist_same)
#6227 by 7

#difference sites
dist_unique<-com_dist_m[!com_dist_m$site1 == com_dist_m$site2,]
dim(dist_unique)
#20342 by 7

#identify duplicated species (i.e. within species comparision) within a site
dist_same$dup<-duplicated(dist_same[,4:7])
dist_same$dup<-gsub('TRUE', 'Between Species', dist_same$dup)
dist_same$dup<-gsub('FALSE', 'Within Species', dist_same$dup)
dist_same$type<-rep('Within a site', nrow(dist_same))

#identify duplicated species (i.e. within species comparision) between sites
dist_unique$dup<-duplicated(dist_unique[,c('species1', 'species2')])
dist_unique$dup<-gsub('TRUE', 'Within Species', dist_unique$dup)
dist_unique$dup<-gsub('FALSE', 'Between Species', dist_unique$dup)
dist_unique$type<-rep('Between Sites', nrow(dist_unique))

#combine back torghe
dist_comb<-rbind(dist_same, dist_unique)

#summarize data
dist_comb_sum<-ddply(dist_comb, c('dup', 'type'), summarize, mean=mean(bc_sim), sd=sd(bc_sim), n=length(bc_sim), se=sd/n)

dist_comb_sum2<-dist_comb_sum
names(dist_comb_sum2)<-c("dup", 'Comparison', 'mean', 'sd', 'n', 'se')


ggplot(dist_comb_sum2, aes(dup, mean, colour=Comparison))+
  geom_point(aes(size=2), position=position_dodge(.5), show.legend = F)+
  geom_errorbar(aes(ymax=mean+(0.05*sd), ymin=mean-(0.05*sd), width=0.2), position=position_dodge(.5))+
  theme_bw()+
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank())+ 
  ylab("Bray-Curtis Dissimilarity")+
  xlab("")+
  scale_colour_manual(values=c("black", "grey"))+
  theme(axis.text=element_text(size=14), axis.title=element_text(size=14))
```

## Test for significance with Welch's t-test w/correction
```
#first, fix data to have multiple cats within a single column
#identify duplicated species (i.e. within species comparision) within a site
dist_same$dup<-duplicated(dist_same[,4:7])
dist_same$dup<-gsub('TRUE', 'Between Species- within sites', dist_same$dup)
dist_same$dup<-gsub('FALSE', 'Within Species- within sites', dist_same$dup)
dist_same$type<-rep('Within a site', nrow(dist_same))

#identify duplicated species (i.e. within species comparision) between sites
dist_unique$dup<-duplicated(dist_unique[,c('species1', 'species2')])
dist_unique$dup<-gsub('TRUE', 'Within Species- between sites', dist_unique$dup)
dist_unique$dup<-gsub('FALSE', 'Between Species- between sites', dist_unique$dup)
dist_unique$type<-rep('Between Sites', nrow(dist_unique))

dist_comb<-rbind(dist_same, dist_unique)

pairwise.t.test(dist_comb$bc_sim, dist_comb$dup, p.adjust.method = 'hochberg')

                              #Between Species- between sites   Between Species- within sites  Within Species- between sites
#Between Species- within sites 9.3e-15                        -                             -                            
#Within Species- between sites 0.832                          < 2e-16                       -                            
#Within Species- within sites  9.1e-14                        0.048                         < 2e-16           
```

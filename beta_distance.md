## Community similarity by distance

```
library(geosphere)
library(vegan)
library(ggplot2)
library(reshape2)

#read in site GPS coordinates
site_gps<-read.delim("bat_mycobiome/site_gps.txt", header=T)

#calculate distance matrix between all GPS coordinates
pairwise_site_distance<-as.data.frame(distm(site_gps[c('long', 'lat')])*0.0006213712)
colnames(pairwise_site_distance)<-site_gps$name
rownames(pairwise_site_distance)<-site_gps$name
pairwise_site_distance$name<-rownames(pairwise_site_distance)

#convert to long format
dist_m<-melt(pairwise_site_distance)
names(dist_m)<-c("site1", 'site2', 'dist')
```

## calculate community similarity

```
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

#read in metadata
meta<-read.delim("bat_mycobiome/uparse_mapping_file.txt", header=T)

#create new data frame just with 'SampleID' and 'area'
site_samp<-meta[,c('SampleID', 'area')]

#add site information to Bray-Curtis similarity values
com_dist_m<-merge(com_dist_m, site_samp, by.x = 'sample1', by.y='SampleID')
names(com_dist_m)<-c("sample1", 'sample2', 'bc_sim', 'site1')

com_dist_m<-merge(com_dist_m, site_samp, by.x = 'sample2', by.y='SampleID')
names(com_dist_m)<-c("sample1", 'sample2', 'bc_sim', 'site1', 'site2')

#combine physical distance with community distance
full_dist<-merge(com_dist_m, dist_m, by=c('site1', 'site2'))
```

## Calculate linear regression and plot data
```
lm(full_dist$bc_sim ~ full_dist$dist)
#slope = 0.0002894, intercept = 0.5290678

#calculate summary stats on a regression
summary(lm(full_dist$bc_sim ~ full_dist$dist))
#F=1511 (1, 26567), p<2e-16, both slope/intercept are sig. @ p<2e-16

#plot data
ggplot(full_dist, aes(dist, bc_sim))+
  geom_point()+
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank())+ 
  theme_bw()+
  geom_abline(aes(slope = 0.0002894, intercept = 0.5290678), size=2)+
  ylab("Bray-Curtis Dissimilarity")+
  xlab("Distance-m")+
  theme(text = element_text(size=14),
        axis.text = element_text(size=14), legend.text=element_text(size=14))
```
  




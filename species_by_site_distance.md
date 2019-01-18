## Community similarity between species within/between sites
```
library(vegan)
library(ggplot2)
library(reshape2)

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

```

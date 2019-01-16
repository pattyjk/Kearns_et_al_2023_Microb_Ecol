## Anlysis of beta and alpha diversity
### Beta diversity
```
#R v. 3.4.4
library(ggplot2)
library(plyr)
library(reshape2)
library(vegan)

#read in meta data
meta<-read.delim("~/bat_mycobiome/uparse_mapping_file.txt", header=T)

#read in mapping file
meta<-read.delim("~/bat_mycobiome/uparse_mapping_file.txt", header=T)

#remove taxonomy column
otu_table<-otu_table[,-165]

#transpose OTU table
otu_table_t<-t(otu_table)

#calculate a MDS plot, unrarefied data
bat.mds<-metaMDS(otu_table_t, distance = 'bray')
#stress=0.192

bat.mds2<-as.data.frame(bat.mds$points)
bat.mds2$SampleID<-row.names(bat.mds2)
bat.mds2<-merge(bat.mds2, meta, by=c('SampleID'))

ggplot(bat.mds2, aes(MDS1, MDS2, colour=ecoregion_iv))+  
  geom_point(aes(size=2))+
  theme_bw()+
  xlab("PC1-26.3%")+
  ylab("PC2-13.7%")+
  theme(text = element_text(size=14),
        axis.text = element_text(size=14), legend.text=element_text(size=14))
```

### Alpha Diversity

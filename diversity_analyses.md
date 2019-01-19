## Anlysis of beta and alpha diversity
### Beta diversity
```
#R v. 3.4.4
library(ggplot2)
library(plyr)
library(reshape2)
library(vegan)

#read in meta data
#setwd("/Users/patty/Dropbox/GitHub/")
meta<-read.delim("bat_mycobiome/uparse_mapping_file.txt", header=T)

#read in OTU table
otu_table<-read.delim("bat_mycobiome/otu_table_tax_R.txt", header=T, row.names=1)

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
names(bat.mds2)<-gsub('area', 'Site', names(bat.mds2))

ggplot(bat.mds2, aes(MDS1, MDS2, colour=Site))+  
  geom_point(aes(size=2))+
  theme_bw()+
  xlab("MDS1")+
  ylab("MDS2")+
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank())+ 
  theme(text = element_text(size=14),
        axis.text = element_text(size=14), legend.text=element_text(size=14))
```

### Alpha Diversity
```
library(plyr)
library(vegan)
library(reshape2)

#read in meta data
meta<-read.delim("bat_mycobiome/uparse_mapping_file.txt", header=T)

#read in OTU table
otu_table<-read.delim("bat_mycobiome/otu_table_tax_R.txt", header=T, row.names=1)
otu_table<-otu_table[,-165]
s16<-t(otu_table)

#rarefy data
library(vegan)
s16<-rrarefy(s16, sample=1200)

#Shannon
s16.shan<-diversity(s16, index='shannon')

#OTUs observed
s16.otus<-rowSums(s16>0)

#Pielou's Evenness
s16.even<-(diversity(s16))/s16.otus

#catenate data into a single frame
s16.div<-as.data.frame(cbind(s16.shan, s16.otus, s16.even))

#add sample IDs
s16.div$SampleID<-row.names(s16.div)

#fix coloumn names
names(s16.div)<-c('Shannon', 'OTUs_Obs', 'Pielous_Even', 'SampleID')

#add metadata
s16.div<-merge(s16.div, meta, by='SampleID')

#summarize data
div.sum<-ddply(s16.div, c("area", "species"), summarize, mean=mean(Shannon), sd=sd(Shannon), n=length(Shannon), se=sd/n)


ggplot(div.sum, aes(species, mean))+
  geom_bar(stat='identity')+
  ylim(0,5)+
  coord_flip()+
  theme_bw()+
  facet_wrap(~area, nrow=1)+
  theme(panel.grid.major = element_blank(), panel.grid.minor = element_blank())+ 
  geom_errorbar(aes(ymax=mean+se, ymin=mean-se), width=0.3, cex=0.2)+
  theme(text = element_text(size=14),
        axis.text = element_text(size=14), legend.text=element_text(size=14))+
  ylab("Shannon Diversity")+
  xlab("")
```

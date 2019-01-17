## Percent occupancy analysis

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
otu_table<-read.delim("bat_mycobiome/otu_table_tax_R.txt", header=T)

#write taxonomy & sample name to new data frames
tax<-otu_table$taxonomy
samp<-otu_table$X

#remove taxonomy and sample name from OTU table
otu_table_n<-otu_table[,c(-166,-1)]

#calculate log OTU abundance
otu_means<-rowMeans(otu_table_n)
otu_means<-as.data.frame(otu_means)

#take log of means
otu_means_log<-log(rowMeans(otu_table_n))
otu_means_log<-as.data.frame(otu_means_log)

#calculate the presence of OTUs
otu_table_binary<-otu_table_n
otu_table_binary[otu_table_binary>0]<-1
otu_samp_count<-rowSums(otu_table_binary)
otu_samp_count<-as.data.frame(otu_samp_count)

#calculate percent occupancy
dim(otu_table_n)
#3487 OTUs, 164 samples

#make a table of OTU counts and mean abundance
per_occu<-cbind(otu_samp_count, otu_means, otu_means_log)
per_occu$per_occ<-((100*(per_occu$otu_samp_count/164)))

#add taxonomy and OTU name to table
per_occu<-cbind(per_occu, tax, samp)


#get OTUs of interest (>55% of sample and log abundance >3)
count55<-which(per_occu$per_occ>55)
count55_p<-per_occu[count55,]
  
count3<-which(per_occu$otu_means_log>3)
count3_p<-per_occu[count3,]

ggplot(per_occu, aes(otu_means_log, per_occ))+
  geom_point(aes(size=.7), colour='grey')+
  geom_point(data=count55_p, aes(otu_means_log, per_occ, size=.7), colour='black')+
  geom_point(data=count3_p, aes(otu_means_log, per_occ, size=.7), colour='black')+
  theme_bw()+
  xlab("Log10 Mean OTU Abundance")+
  ylab("Percent Occupancy")+
  theme(text = element_text(size=14),
        axis.text = element_text(size=14), legend.text=element_text(size=14))


#make a data frame of 'core' OTUs
core_otus<-rbind(count3_p, count55_p)

#write core OTUs to table
write.table(core_otus, "bat_mycobiome/core_otus.txt", row.names=F, quote=F, sep='/t')
```

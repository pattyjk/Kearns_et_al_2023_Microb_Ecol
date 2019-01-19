## Analyis of the taxonomic composition of bats
```
#QIIME 1.9.1
summarize_taxa.py -i out_table_tax2.biom -o tax_sum
```

## Plot in R
```
#R v. 3.4.4
library(ggplot2)
library(reshape2)
library(plyr)

#read in class level taxonomy
class_tax<-read.delim("bat_mycobiome/tax_sum/otu_table_tax_L3_R.txt", header=T, row.names = 1)

#read in meta data
meta<-read.delim("bat_mycobiome/uparse_mapping_file.txt", header=T)

#get row sums
class_tax$sum<-rowSums(class_tax)
class_tax<-class_tax[order(class_tax$sum, decreasing=T) , ]

#keep only top 10 classes
class_tax<-class_tax[1:10,]

#remove sum column
class_tax<-as.data.frame(class_tax[,-grep('sum', names(class_tax))])

#get 'others' category (things not in top 10)
others<-1-colSums(class_tax)
class_tax<-rbind(class_tax, others)
row.names(class_tax)[11]<-'Others'

#add back in taxonomy
class_tax$Class<-row.names(class_tax)

#melt data
class_m<-melt(class_tax)

#add meta data
class_m<-merge(class_m, meta, by.x='variable', by.y='SampleID')

#take means of site/species
class_sum<-ddply(class_m, c("Class", "species", 'area'), summarize, mean=mean(value), sd=sd(value), n=length(value), se=sd/n)

#make pallette
col_vector<-c('#e6194b', '#3cb44b', '#ffe119', '#4363d8', '#f58231', '#911eb4', '#46f0f0', '#f032e6', '#bcf60c', '#fabebe', '#008080', '#e6beff', '#9a6324', '#fffac8', '#800000', '#aaffc3', '#808000', '#ffd8b1', '#000075', '#808080', '#ffffff', '#000000')

#plot it
ggplot(class_sum, aes(species, mean, fill=Class))+
  geom_bar(stat='identity')+
  xlab("")+
  ylab("Mean Relative Abundance")+
  theme_bw()+
  scale_fill_manual(values=col_vector)+
  facet_wrap(~area, scales = 'free_x')+
  coord_flip()+
  theme(text = element_text(size=14),
        axis.text = element_text(size=14), legend.text=element_text(size=14))
```

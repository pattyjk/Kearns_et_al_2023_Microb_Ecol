## Anlysis of alpha diversity

### Alpha Diversity
```
library(plyr)
library(vegan)
library(ggplot2)
library(reshape2)
library(rstanarm)

#read in meta data
meta<-read.delim("bat_mycobiome/uparse_mapping_file_temp.txt", header=T)

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

### Examine correlation between diversity and meta data and location
```
#read in lat/long data
latlong<-read.delim("bat_mycobiome/site_gps.txt", header=T)

#merge lat/long to diversity data
s16.div<-merge(s16.div, latlong, by.x='area', by.y='name')

cor.test(s16.div$lat, s16.div$OTUs_Obs)
#p=0.00024, Rho=0.28

cor.test(s16.div$lat, s16.div$Shannon)
#Rho=0.185, p=0.0181

ggplot(s16.div, aes(lat, Shannon))+
  geom_point()+
  theme_bw()+
  ylab("Shannon Diversity")+
  xlab("Latitude")+
  geom_smooth(method='lm')+
  facet_wrap(~species, scale='free')
 ```
 
 
 ### Linear mixed effects model to explain drivers of diversity
 ```
#run glmer bayesian model
bat.glmer<- stan_glmer(OTUs_Obs ~ lat + (1|species) + (1|area) + (1|sex) + (1|cave_or_surface) + (1|Feeding.Flight.behavior) + (1|Diet), data=s16.div, family= gaussian)
bat.glmer.sum<-as.data.frame(summary(bat.glmer))
View(bat.glmer.sum)
write.table(bat.glmer.sum, 'bat_glmer_sum.txt', sep='\t', row.names=F, quote=F)
 ```
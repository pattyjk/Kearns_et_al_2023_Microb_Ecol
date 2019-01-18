## FunGuild analsis
```
#get latest version, clones on 1/15/19
git clone https://github.com/UMNFuN/FUNGuild.git

#if recieving 'no module names request' error use: "pip install requests"

#move OTU table to funguild folder
cp otu_table_tax.txt ~/FUNGuild/otu_table_tax2.txt
cd FUNGuild

#need to remove first line and # and space beteween OTU ID, will do manually

#run FunGuild
python Guilds_v1.1.py -otu '/home/pattyjk/FUNGuild/otu_table_tax2.txt' -db fungi -m -u
```

## Analysis in ARRR
```
#R v. 3.4.4
#read in core OTUs
core_otus<-read.delim("bat_mycobiome/core_otus.txt", header=T)

#read in FunGuild data
funguild<-read.delim("bat_mycobiome/FunGuild_analysis/otu_table_tax2.guilds.txt", header=T)

#extract stuff of interest
funguild2<-funguild[,c(1,178:185)]

#merge funguild data to core OTUs
core_funguild<-merge(core_otus, funguild2, by.x='samp', by.y='OTUID')

#write data to file
write.table(core_funguild, 'core_otus_funguild.txt', row.names=F, quote=F, sep='\t')
```

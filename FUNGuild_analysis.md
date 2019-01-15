
#get latest version
git clone https://github.com/UMNFuN/FUNGuild.git

#if recieving 'no module names request' error use: "pip install requests"

#move OTU table to funguild folder
cp otu_table_tax.txt ~/FUNGuild/otu_table_tax2.txt
cd FUNGuild

#need to remove first line and # and space beteween OTU ID, will do manually

#run FunGuild
python Guilds_v1.1.py -otu '/home/pattyjk/FUNGuild/otu_table_tax2.txt' -db fungi -m -u
```

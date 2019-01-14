# ITS sequence processing in UPARSE and QIIME

## Dereplicate sequences
```
./usearch64 -fastx_uniques bats_its_catenated.fna -fastaout uniques_combined_merged.fa -sizeout
```

## Remove Singeltons
```
./usearch64 -sortbysize uniques_combined_merged.fa -fastaout nosigs_uniques_combined_merged.fa -minsize 2
```

## Precluster Sequences
```
./usearch64 -cluster_fast nosigs_uniques_combined_merged.fa -centroids denoised_nosigs_uniques_combined_merged.fa -id 0.9 -maxdiffs 5 -abskew 10 -sizein -sizeout -sort size
```

## Reference-based OTU picking against UNITE v. 7
```
./usearch64 -usearch_global denoised_nosigs_uniques_combined_merged.fa -id 0.97 -db /media/pattyjk/Elements/UNITE/sh_refs_qiime_ver7_97_s_01.12.2017.fasta  -strand plus -uc ref_seqs.uc -dbmatched closed_reference.fasta -notmatched failed_closed.fa
```

## Sort by size and then de novo OTU picking on sequences that failed to hit GreenGenes
```
./usearch64 -sortbysize failed_closed.fa -fastaout sorted_failed_closed.fa

./usearch64 -cluster_otus sorted_failed_closed.fa -minsize 2 -otus denovo_otus.fasta -relabel OTU_dn_ -uparseout denovo_out.up
```

## Combine the rep sets between de novo and reference-based OTU picking
```
cat closed_reference.fasta denovo_otus.fasta > full_rep_set.fna
```

## Map rep_set back to pre-dereplicated sequences and make OTU tables
```
./usearch64 -usearch_global bats_its_catenated.fna -db full_rep_set.fna  -strand plus -id 0.97 -uc OTU_map.uc -otutabout OTU_table.txt -biomout OTU_jsn.biom
```

## Assigng taxonomy with RDP classifier and add to OTU table
```
#QIIME 1.9.1
# RDP classifier 

assign_taxonomy.py -i full_rep_set.fna -o taxonomy -r '/media/pattyjk/Elements/UNITE/sh_refs_qiime_ver7_97_s_01.12.2017.fasta' -t '/media/pattyjk/Elements/UNITE/sh_taxonomy_qiime_ver7_97_s_01.12.2017.txt' -m rdp

#add taxonomy to OTU table
biom add-metadata -i OTU_jsn.biom -o otu_table_tax.biom --observation-metadata-fp=taxonomy/full_rep_set_tax_assignments.txt --sc-separated=taxonomy --observation-header=OTUID,taxonomy
```


```



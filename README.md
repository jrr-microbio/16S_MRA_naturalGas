## 

QIIME2 Repository: Natural Gas Reservoir
------------

The role of this repository is to store the QIIME2 code used for the publication titled: Prokaryotic Community Profiling from Pore Fluids Collected from a Natural Gas Reservoir

## Abstract
Environmental gene profiling can provide valuable insights into low biomass microbial communities inhabiting subsurface environments. To access the prokaryotic diversity of an active natural gas storage field, 16S rRNA gene sequencing was performed on formation water to provide insights into the taxonomic composition of microbial groups potentially capable of metabolizing stored fuel gases in the geologic reservoir. 

```
#Install qiime2
mamba env create \
  --name qiime2-amplicon-2025.7 \
  --file https://raw.githubusercontent.com/qiime2/distributions/refs/heads/dev/2025.7/amplicon/released/qiime2-amplicon-ubuntu-latest-conda.yml

mamba activate qiime2-amplicon-2025.7

###################
###################
  #Let's try this with paired end reads
###################
###################

#Import manifest with reads
  qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path ./_ASV_16S_manifest.csv \
  --output-path ./paired-end-demux.qza \
  --input-format PairedEndFastqManifestPhred33

#Visualize quality
  qiime demux summarize \
  --i-data paired-end-demux.qza \
  --o-visualization demux.qzv

#cutadapt is being weird here...but i know the primers so lets just dada2 with the length of the primers.
#qiime cutadapt trim-paired \
#  --i-demultiplexed-sequences paired-end-demux.qza \
#  --p-cores 1 \
#  --p-adapter-f GTGYCAGCMGCCGCGGTAA \ #19bp
#  --p-adapter-r GGACTACNVGGGTWTCTAAT \ #20bp
#  --o-trimmed-sequences trimmed-sequences.qza

#Visualize quality after primer removal
#  qiime demux summarize \
#  --i-data trimmed-sequences.qza \
#  --o-visualization demux_trimmed.qzv

#Trim and denoise - use method consensus. Using chimera method consensus based on this: https://forum.qiime2.org/t/dada2-chimera-filtering-and-beyond/8685/2

qiime dada2 denoise-paired \
--i-demultiplexed-seqs paired-end-demux.qza \
--p-trim-left-f 19 \
--p-trim-left-r 20 \
--p-trunc-len-f 0 \
--p-trunc-len-r 0 \
--p-n-threads 10 \
--p-trunc-q 2 \
--p-chimera-method consensus \
--o-representative-sequences rep-seqs-dada2.qza \
--o-table table-dada2.qza \
--o-denoising-stats stats-dada2.qza

#Now look at how denoising changed results.
  qiime metadata tabulate \
  --m-input-file stats-dada2.qza \
  --o-visualization stats-dada2.qzv

#Export the otu table.
  qiime tools export \
  --input-path table-dada2.qza \
  --output-path exported-feature-table

#Convert biom to tsv.
biom convert --to-tsv -i ./exported-feature-table/feature-table.biom -o final_ASV_table.tsv

qiime tools export \
  --input-path rep-seqs-dada2.qza \
  --output-path exported-rep-seqs

###############
###############
###############
# Now let's classify with SILVA - i already have a trained classifier so just running it.
###############
###############
###############

#Can be downloaded from: https://www.arb-silva.de/current-release/QIIME2/2025.7/SSU/full-length/uniform

qiime feature-classifier classify-sklearn \
  --i-classifier SILVA138.2_SSURef_NR99_uniform_classifier_full-length.qza \
  --i-reads rep-seqs-dada2.qza \
  --o-classification classified_rep-seqs_wSILVA_taxonomy.qza

qiime tools export \
--input-path classified_rep-seqs_wSILVA_taxonomy.qza \
--output-path exported_taxonomy


qiime krona collapse-and-plot \
--i-table table-dada2.qza \
--i-taxonomy /classified_rep-seqs_wSILVA_taxonomy.qza \
--o-krona-plot krona.qzv
```

<img width="1400" height="1262" alt="taxonomy_order-1" src="https://github.com/user-attachments/assets/b2e68bef-2f73-46f2-b711-57d952ae6552" />
<img width="1088" height="1245" alt="Screenshot 2025-10-25 at 10 35 10 AM" src="https://github.com/user-attachments/assets/15b10b7b-1010-4333-bdf0-324b029dc996" />


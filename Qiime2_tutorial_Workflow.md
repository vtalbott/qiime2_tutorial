#### Import demultiplexed sequence datausing manifest file tutorial

1. Get manifests, make sure you are in the qiime2_tutorial directory
```
cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/manifest_run2.txt .

cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/manifest_run3.txt .
```

2. Get demultiplexed sequences 
```
cp -r /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/reads_run2 .

cp -r /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/reads_run3 .

```

3. Import with manifest file
```
qiime tools import \--type "SampleData[PairedEndSequencesWithQuality]" \--input-format PairedEndFastqManifestPhred33V2 \--input-path manifest_run2.txt \--output-path demux_run2.qza
```

4. Visualize file
```
qiime demux summarize \--i-data demux_run2.qza \--o-visualization demux_run2.qzv
```

5. Import with manifest file
```
qiime tools import \--type "SampleData[PairedEndSequencesWithQuality]" \--input-format PairedEndFastqManifestPhred33V2 \--input-path manifest_run3.txt \--output-path demux_run3.qza
```

6. Visualize file
```
qiime demux summarize \--i-data demux_run3.qza \--o-visualization demux_run3.qzv
```

End of demultiplexing


#### Denoising Sequence Data with DADA2 tutorial

Denoise samples to ASV's using DADA2 to create a feature table - run 2

```
qiime dada2 denoise-paired \--i-demultiplexed-seqs demux_run2.qza \--p-trunc-len-f 150 \--p-trunc-len-r 150 \--p-n-threads 6 \--o-table table_run2.qza \--o-representative-sequences seqs_run2.qza \--o-denoising-stats dada2_stats_run2.qza
```

Review outputs for run 2 using the qiime metadata tabulate command
```
qiime metadata tabulate \--m-input-file dada2_stats_run2.qza \--o-visualization dada2_stats_run2.qzv
```

Denoise samples to ASV’s using DADA2 to create a feature table - run 3
```
qiime dada2 denoise-paired \--i-demultiplexed-seqs demux_run3.qza \--p-trunc-len-f 150 \--p-trunc-len-r 150 \--p-n-threads 6 \--o-table table_run3.qza \--o-representative-sequences seqs_run3.qza \--o-denoising-stats dada2_stats_run3.qza
```

Review denoising statistics for run 3 using the qiime metadata tabulate command
```
qiime metadata tabulate \--m-input-file dada2_stats_run3.qza \--o- visualization dada2_stats_run3.qzv
```

Merge run 2 and run 3 denoising output tables and summarize
```
qiime feature-table merge \--i-tables table_run2.qza \--i-tables table_run3.qza \--o-merged-table table.qza

  
qiime feature-table summarize \--i-table table.qza \--o-visualization table.qzv \--m-sample-metadata-file metadata_q2_workshop.txt
```

Merge run 2 and run 3 denoising output representative sequences and summarize
```
qiime feature-table merge-seqs \--i-data seqs_run2.qza \--i-data seqs_run3.qza \--o-merged-data seqs.qza

  
qiime feature-table tabulate-seqs \--i-data seqs.qza \--o-visualization seqs.qzv

```


End of Denoising tutorial

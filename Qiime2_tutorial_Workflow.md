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
qiime metadata tabulate \--m-input-file dada2_stats_run3.qza \--o-visualization dada2_stats_run3.qzv
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

## End of Day 1



## Day 2

#### Taxonomy Tutorial

Start Interactive session
1. Request interactive session with workshop reservation
```
sinteractive --reservation=microbiome --ntasks=4 --time=04:00:00**
```

2. Activate Qiime2
```
module purge  
module load qiime2/2024.10_amplicon
```

Grabbing the taxonomic classifier (GreenGenes2)
```
# First navigate to your qiime2_tutorial directory

cd /scratch/alpine/USER@colostate.edu/qiime2_tutorial

# Now copy the provided GG2 taxonomic classifier

cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/[2024.09.backbone.v4.nb](http://2024.09.backbone.v4.nb).qza .
```

Taxonomically classifying our feature table
```
qiime feature-classifier classify-sklearn \--i-reads seqs.qza \--i-classifier 2024.09.backbone.v4.nb.qza \--o-classification taxonomy_gg2.qza
```

Checking taxonomy associated with sequences
```
qiime metadata tabulate \--m-input-file taxonomy_gg2.qza \--o-visualization taxonomy_gg2.qzv
```

Taxonomy Barcharts
```
qiime feature-table group \--i-table table.qza \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-column type_days \--p-mode mean-ceiling \--p-axis sample \--o-grouped-table table_type_days.qza
```

Taxonomy Barcharts
```
qiime taxa barplot \--i-table table_type_days.qza \--i-taxonomy taxonomy_gg2.qza \--o-visualization taxa_barplot_type_days.qzv
```

Removing chloroplast/mitochondrial reads
```
qiime taxa filter-table \--i-table table_type_days.qza \--i-taxonomy taxonomy_gg2.qza \--p-exclude mitochondria,chloroplast,sp004296775 \--o-filtered-table table_type_days_nomitochloro.qza

  

# The below command we will run now, but it will be used later in the R section

qiime taxa filter-table \--i-table table.qza \--i-taxonomy taxonomy_gg2.qza \--p-exclude mitochondria,chloroplast,sp004296775 \--o-filtered-table table_nomitochloro.qza

```

Taxa Barchart with no mitochondria/chloroplast reads
```
qiime taxa barplot \--i-table table_type_days_nomitochloro.qza \--i-taxonomy taxonomy_gg2.qza \--o-visualization taxa_barplot_type_days_nomitochloro.qzv
```

End of Taxonomy Tutorial


#### Quality Control Tutorial

Look at control reads per sample
```
qiime feature-table summarize \--i-table table_nomitochloro.qza \--o-visualization table_nomitochloro.qzv \--m-sample-metadata-file metadata_q2_workshop.txt
```

Taxonomy Barcharts
```
# First filter samples

qiime feature-table filter-samples \--i-table table.qza \--m-metadata-file metadata_q2_workshop.txt \--p-where "[sample_type]='control'" \--o-filtered-table table_controls.qza

```

Taxonomy Barcharts
```
# Create taxa barplot with our table with just controls

qiime taxa barplot \--i-table table_controls.qza \--i-taxonomy taxonomy_gg2.qza \--m-metadata-file metadata_q2_workshop.txt \--o-visualization taxa_barplot_controls.qzv

```

End of Quality Control Tutorial


#### Identifying an Even Sampling Depth for use in Core Metrics Tutorial i.e. rarefaction

Alpha Rarefaction 
- This will provide us with a plot of different diversity metrics and different rarefaction depths
```
qiime diversity alpha-rarefaction \--i-table table_nomitochloro.qza \--m-metadata-file metadata_q2_workshop.txt \--p-max-depth 5500 \--o-visualization alpha_rarefaction_curves.qzv**
```


End of Rarefaction


#### Phylogenetic Tree Construction Tutorial 

Get a reference tree!
```
cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/2022.10.backbone.sepp-reference.qza .**
```
- Reference tree available through Greengenes 2 website

Place you sequences onto the tree
- takes ~1.5 hours to run! Run on your own time if you'd like!
```
qiime fragment-insertion sepp \--i-representative-sequences seqs.qza \--i-reference-database 2022.10.backbone.sepp-reference.qza \--o-tree tree_gg2.qza \--o-placements tree_gg2_placements.qza \--p-threads 4
```

Get a copy of an already built tree!
- This tree will only work with this dataset. You will need to build a new tree for each new dataset. 
```
cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/tree_gg2.qza .
```

End of Phylogenetic Tree Construction Tutorial


#### α Diversity Metrics Tutorial 

Core Metrics
```
qiime diversity core-metrics-phylogenetic \--i-table ./dada2_table.qza \--i-phylogeny ./tree.qza \--m-metadata-file ./metadata.tsv \--p-sampling-depth 2000 \--output-dir ./core-metrics-results
```

Faith's Phylogenetic Diversity
```
qiime diversity alpha-group-significance \--i-alpha-diversity ./core-metrics-results/faith_pd_vector.qza \--m-metadata-file ./metadata.tsv \--o-visualization ./core-metrics-results/faiths_pd_statistics.qzv
```

Pielou's Evenness
```
qiime diversity alpha-group-significance \--i-alpha-diversity ./core-metrics-results/evenness_vector.qza \--m-metadata-file ./metadata.tsv \--o-visualization ./core-metrics-results/evenness_statistics.qzv
```

End of alpha diversity metrics tutorial 


#### β diversity Visualizations Tutorial

PERMANOVA - tests for differences between group Unweighted Unifrac
```
qiime diversity beta-group-significance \--i-distance-matrix core_metrics/unweighted_unifrac_distance_matrix.qza \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-column sample_type \--o-visualization core_metrics/unweighted_unifrac_sample_type_significance.qzv
```

PERMANOVA - tests for differences between group Weighted Unifrac
```
qiime diversity beta-group-significance \--i-distance-matrix core_metrics/weighted_unifrac_distance_matrix.qza \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-column sample_type \--o-visualization core_metrics/weighted_unifrac_sample_type_significance.qzv
```

Adonis - test for multiple effects
```
qiime diversity adonis \--i-distance-matrix core_metrics/weighted_unifrac_distance_matrix.qza \--m-metadata-file metadata_q2_workshop.txt \--o-visualization core_metrics/weighted_adonis.qzv \--p-formula add_0c+facility+sample_type
```


PERMANOVA - tests for differences in soil samples
- Filter to soil and rerun coremetrics
```
# Filter
qiime feature-table filter-samples \--i-table table_nomitochloro.qza \--m-metadata-file metadata_q2_workshop.txt \--p-where "[sample_type]='soil'" \--o-filtered-table table_nomitochloro_soil.qza

```

```  
# Rerun coremetrics
qiime diversity core-metrics-phylogenetic \--i-table table_nomitochloro_soil.qza \--i-phylogeny tree_gg2.qza \--m-metadata-file metadata_q2_workshop.txt \--p-sampling-depth 1500 \--output-dir core_metrics_soil
```

PERMANOVA - tests for differences in soil samples
```
qiime diversity beta-group-significance \--i-distance-matrix core_metrics_soil/unweighted_unifrac_distance_matrix.qza \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-column facility \--o-visualization core_metrics_soil/unweighted_unifrac_facility_significance.qzv
```

End of Beta Diversity
## End of Day 2


## Day 3 

#### Differential Abundance with ANCOM-BC Tutorial

1. Request interactive session with workshop reservation 
```
sinteractive --reservation=microbiome --ntasks=4 --time=04:00:00
```

2. Activate Qiime2 and change into your qiime2_tutorial directory
```
module purge  
module load qiime2/2024.10_amplicon

cd /scratch/alpine/USER@colostate.edu/qiime2_tutorial
```

Filter out samples with low frequency and filter out low abundance ASVs
```
qiime feature-table filter-samples \--i-table table_nomitochloro.qza \--p-min-frequency 1500 \--o-filtered-table table_nomitochloro_1500.qza

  

qiime feature-table filter-features \--i-table table_nomitochloro_1500.qza \--p-min-frequency 50 \--p-min-samples 4 \--o-filtered-table table_nomitochloro_1500_abund.qza

```

Collapse feature table to the species level
```
qiime taxa collapse \--i-table table_nomitochloro_1500_abund.qza \--i-taxonomy taxonomy_gg2.qza \--p-level 7 \--o-collapsed-table table_nomitochloro_1500_abund_L7.qza
```

Run ANCOM on sample type - test between sample types
```
cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/QIIME2/metadata_q2_workshop_noECs.txt .

  

qiime composition ancombc \--i-table table_nomitochloro_1500_abund_L7.qza \--m-metadata-file metadata_q2_workshop_noECs.txt \--p-formula 'sample_type' \--o-differentials ancombc_sample_type.qza
```

Visualize ANCOM results
```
qiime composition tabulate \--i-data ancombc_sample_type.qza \--o-visualization ancombc_sample_type.qzv

  

qiime composition da-barplot \--i-data ancombc_sample_type.qza \--p-significance-threshold 0.001 \--p-label-limit 5000 \--p-level-delimiter ";" \--o-visualization da_barplot_sample_type.qzv
```

Run ANCOM on sample type - test between facilities
```
qiime composition ancombc \--i-table table_nomitochloro_1500_abund_L7.qza \--m-metadata-file metadata_q2_workshop_noECs.txt \--p-formula 'facility' \--o-differentials ancombc_facility.qza
```

Visualize ANCOM results
```
qiime composition tabulate \--i-data ancombc_facility.qza \--o-visualization ancombc_facility.qzv

  

qiime composition da-barplot \--i-data ancombc_facility.qza \--p-significance-threshold 0.001 \--p-label-limit 5000 \--p-level-delimiter ";" \--o-visualization da_barplot_facility.qzv
```

End of ANCOM-BC tutorial 


#### Machine Learning Tutorial

Can we predict the facility where a sample was collected?
```
qiime sample-classifier classify-samples \--i-table table_nomitochloro_1500_abund_L7.qza \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-column facility \--p-random-state 123 \--p-n-jobs 1 \--output-dir sample_classifier_results_facility
```

What features were the most important?
- Displays (normalized) feature abundances per sample
```
qiime sample-classifier heatmap \--i-table table_nomitochloro_1500_abund_L7.qza \--i-importance sample_classifier_results_facility/feature_importance.qza \--m-sample-metadata-file metadata_q2_workshop.txt \--m-sample-metadata-column facility \--p-group-samples \--p-feature-count 100 \--o-heatmap sample_classifier_results_facility/heatmap_100_features.qzv \--o-filtered-table sample_classifier_results_facility/filtered_table_100_features.qza
```

Can we predict the ADD (time +temp) of samples at time of collection?
- Table should be rarefied
- We will use the table that is also collapsed on species but you can also use an ASV table
```
qiime sample-classifier regress-samples \--i-table table_nomitochloro_1500_abund_L7.qza \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-column add_0c \--p-estimator RandomForestRegressor \--p-n-estimators 20 \--p-random-state 123 \--output-dir sample_regressor_results_ADD
```

What microbes are the most important for predictions?
```
qiime metadata tabulate \--m-input-file sample_regressor_results_ADD/feature_importance.qza \--o-visualization sample_regressor_results_ADD/feature_importance.qzv
```

End of machine learning tutorial 


#### Longitudinal Tutorial 

PCoA-based longitudinal analysis
```
qiime longitudinal volatility \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-file core_metrics/weighted_unifrac_pcoa_results.qza \--p-state-column add_0c \--p-individual-id-column host_subject_id \--p-default-group-column 'sample_type' \--p-default-metric 'Axis 2' \--o-visualization pc_vol_sample_type.qzv
```

Distance based longitudinal analysis - calculate first distances
```
qiime longitudinal first-distances \--i-distance-matrix core_metrics/weighted_unifrac_distance_matrix.qza \--m-metadata-file metadata_q2_workshop.txt \--p-state-column add_0c \--p-individual-id-column host_subject_id_sample_type \--p-baseline 0 \--o-first-distances from_first_wunifrac.qza
```


Distance-based longitudinal analysis - visualize first-distances
```
qiime longitudinal volatility \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-file from_first_wunifrac.qza \--p-state-column add_0c \--p-individual-id-column host_subject_id \--p-default-metric Distance \--p-default-group-column 'sample_type' \--o-visualization from_first_wunifrac_vol.qzv
```


Linear mixed effects (LME) model
```
qiime longitudinal linear-mixed-effects \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-file from_first_wunifrac.qza \--p-state-column add_0c \--p-individual-id-column host_subject_id \--p-formula "Distance ~ add_0c + facility + sample_type" \--o-visualization from_first_wunifrac_lme_formula.qzv
```

Alpha Diversity - Volatility 
```
qiime longitudinal volatility \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-file core_metrics/shannon_vector.qza \--p-default-metric shannon_entropy \--p-default-group-column sample_type \--p-state-column add_0c \--p-individual-id-column host_subject_id \--o-visualization sample_type_shannon_volatility.qzv
```

Alpha Diversity - Linear mixed effects (LME) model
```
qiime longitudinal linear-mixed-effects \--m-metadata-file metadata_q2_workshop.txt \--m-metadata-file core_metrics/shannon_vector.qza \--p-state-column add_0c \--p-individual-id-column host_subject_id \--p-formula "shannon_entropy ~ add_0c + facility + sample_type" \--o-visualization shannon_lme_formula.qzv
```


End of Longitudinal Tutorial 


#### Generating Publication Quality Plots in R

Exporting alpha diversity from QIIME2
```
qiime tools export \--input-path core_metrics/evenness_vector.qza \--output-path output_for_R

  

mv output_for_R/alpha-diversity.tsv output_for_R/evenness_vector.tsv

  

qiime tools export \--input-path core_metrics/faith_pd_vector.qza \--output-path output_for_R

  

mv output_for_R/alpha-diversity.tsv output_for_R/faith_pd_vector.tsv

```

Exporting Alpha diversity from QIIME2
```
qiime tools export \--input-path core_metrics/observed_features_vector.qza \--output-path output_for_R

  

mv output_for_R/alpha-diversity.tsv output_for_R/observed_features_vector.tsv

  

qiime tools export \--input-path core_metrics/shannon_vector.qza \--output-path output_for_R

  

mv output_for_R/alpha-diversity.tsv output_for_R/shannon_vector.tsv
```


Exporting Beta diversity from QIIME2
```
qiime tools export \--input-path core_metrics/bray_curtis_distance_matrix.qza \--output-path output_for_R

  

mv output_for_R/distance-matrix.tsv output_for_R/bray_curtis_distance_matrix.tsv

  

qiime tools export \--input-path core_metrics/jaccard_distance_matrix.qza \--output-path output_for_R

  

mv output_for_R/distance-matrix.tsv output_for_R/jaccard_distance_matrix.tsv

```


Exporting beta diversity from QIIME2
```
qiime tools export \--input-path core_metrics/unweighted_unifrac_distance_matrix.qza \--output-path output_for_R

  

mv output_for_R/distance-matrix.tsv output_for_R/unweighted_unifrac_distance_matrix.tsv

  

qiime tools export \--input-path core_metrics/weighted_unifrac_distance_matrix.qza \--output-path output_for_R

  

mv output_for_R/distance-matrix.tsv output_for_R/weighted_unifrac_distance_matrix.tsv
```

Exporting taxonomy from QIIME2
```
qiime feature-table transpose \--i-table table_nomitochloro.qza \--o-transposed-feature-table table_nomitochloro_transposed.qza

  

qiime metadata tabulate \--m-input-file table_nomitochloro_transposed.qza \--m-input-file seqs.qza \--m-input-file taxonomy_gg2.qza \--o-visualization tabulated_results.qzv
```

Exporting ANCOM-BC output from QIIME2
```
qiime tools export \--input-path ancombc_sample_type.qza \--output-path output_for_R/ancombc/sample_type

  

qiime tools export \--input-path ancombc_facility.qza \--output-path output_for_R/ancombc/facility
```

Exporting machine learning feature importances from QIIME2 - sample type
```
qiime feature-table transpose \--i-table sample_classifier_results_sample_type/filtered_table_100_features.qza \--o-transposed-feature-table sample_classifier_results_sample_type/filtered_table_100_features_transposed.qza

  

qiime metadata tabulate \--m-input-file sample_classifier_results_sample_type/filtered_table_100_features_transposed.qza \--m-input-file sample_classifier_results_sample_type/feature_importance.qza \--o-visualization sample_classifier_results_sample_type/table_100_feature_importances.qzv
```

Exporting machine learning feature importances from QIIME2 - facility
```
qiime feature-table transpose \--i-table sample_classifier_results_facility/filtered_table_100_features.qza \--o-transposed-feature-table sample_classifier_results_facility/filtered_table_100_features_transposed.qza

  

qiime metadata tabulate \--m-input-file sample_classifier_results_facility/filtered_table_100_features_transposed.qza \--m-input-file sample_classifier_results_facility/feature_importance.qza \--o-visualization sample_classifier_results_facility/table_100_feature_importances.qzv
```

Create R folders and get the R project and script
```
# Within your qiime2_tutorial folder:

  

mkdir R_output_data

mkdir R_figures

  

cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/R/visualizations.Rmd .

cp /pl/active/courses/2025_summer/CSU_2025/q2_workshop_final/R/R.Rproj .

```

## End of Day 3


```
qiime metadata tabulate \--m-input-file dada2_stats_run3.qza \--o- visualization dada2_stats_run3.qzv
```

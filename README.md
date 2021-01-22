# Deprecated repo
Don't use repo! There is code and a pipeline that is much easier to use.
Look at https://github.com/oscarlr-TRs/PacMonSTR and https://github.com/oscarlr-TRs/PacMonSTR-merge

# Genotype-TRs
Genotyping TRs using MsPAC and PacMonSTR

## Code and tools needed
```
Tools:
1. pbmm2

Code:
1. https://github.com/oscarlr/tr/blob/master/python/extract_tr_loci_from_read_alignment.py
2. https://github.com/oscarlr/TR-DP-algorithm/blob/master/pacMonStr_V1.py

```

## Step 1. Align subreads to reference
```
ref=
bam=
sample=
submitjob 24 -c 20 -m 40 -q premium \
  pbmm2 align \
  ${ref} \
  ${bam} \
  -j 20 \
  -J 20 \
  align_to_hg38/${sample}_subreads_to_hg38.sorted.bam \
  --sort
```

## Step 2. Extract reads overlapping TRs
```
sample=
python /sc/hydra/work/rodrio10/software/in_github/tr/python/extract_tr_loci_from_read_alignment.py \
  align_to_hg38/${sample}_subreads_to_hg38.sorted.bam \
  repeats.bed \
  /sc/hydra/projects/sharpa01a/rodrio10-hydra/databases/references/hg38/chr_no_alts/pbmm2_index/hg38.fa \
  extract_sequence/${sample}/
  
$ cat repeats.bed
chr16	58516125	58516152	5	GGGGC	5.6
chr10	11742312	11742382	5	GGGGC	14.2

```

## Step 3. Genotype TRs using PacMonSTR
```
sample=
python /sc/hydra/work/rodrio10/software/in_github/TR-DP-algorithm/pacMonStr_V1.py \
  extract_sequence/${sample}/query.fasta \
  extract_sequence/${sample}/prefix.fasta \
  extract_sequence/${sample}/suffix.fasta \
  extract_sequence/${sample}/tr.fasta > \
  genotype_trs/${sample}/${sample}.trs  
```

## Step 4. Collapse output from PacMonSTR
```
# input arguments
# 1. Repeats that were genotyped
# 2. Haplotype 1 genotyped TRs
# 3. Haplotype 2 genotyped TRs
# 4. Yes or No; Filter TRs based on alignment scores
# 5. Float from 0 to 1. The alignment score to fitler on
python /sc/hydra/work/rodrio10/software/in_github/tr/python/merge_trs_from_phased_reads_from_controls.py \
  repeats.bed \
  extract_sequence/${sample}/${sample}.trs \
  extract_sequence/${sample}/${sample}.trs \
  No \
  .8 > genotype_trs/${sample}/${sample}.trs.all
```

## Step 5. Get Z-scores 
```
sample=
python /sc/hydra/work/rodrio10/projects/TRs/Famalial_ataxia_cases/bin/python/calculate_zscore_against_control.py \
  controls.bed \
  genotype_trs/${sample}/${sample}.trs.all > \
  compute_z_scores/${sample}.bed
```

## Step 6. Compare TRs against controls
```
bedtools intersect -b genotype_trs/${sample}/${sample}.trs.all -a controls.bed >> get_copies_from_other_samples.txt
```

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

## Step 4. Compare TRs against controls
```
bedtools intersect -b repeats.bed -a controls.bed >> get_copies_from_other_samples.txt
```

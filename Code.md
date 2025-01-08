# RNA Virus
## Trimmomatic, SortmeRNA, MEGAHIT
```
#!/bin/bash
#SBATCH

## trim
R1=sample.R1.fq.gz
R2=sample.R2.fq.gz
R1_T=sample.R1_trimmed.fq.gz
R1_T_U=sample.R1_trimmed_U.fq.gz
R2_T=sample.R2_trimmed.fq.gz
R2_T_U=sample.R2_trimmed_U.fq.gz
trimmomatic PE -threads 40  $R1 $R2 $R1_T $R1_T_U $R2_T $R2_T_U ILLUMINACLIP:/datanode03/liupf/common_files/TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36

## sortmerna
conda activate sortmerna
sortmerna \
 --ref /datanode03/liupf/software_lpf/sortmeRNA_db/rfam-5.8s-database-id98.fasta \
 --ref /datanode03/liupf/software_lpf/sortmeRNA_db/rfam-5s-database-id98.fasta \
 --ref /datanode03/liupf/software_lpf/sortmeRNA_db/silva-arc-16s-id95.fasta \
 --ref /datanode03/liupf/software_lpf/sortmeRNA_db/silva-arc-23s-id98.fasta \
 --ref /datanode03/liupf/software_lpf/sortmeRNA_db/silva-bac-16s-id90.fasta \
 --ref /datanode03/liupf/software_lpf/sortmeRNA_db/silva-bac-23s-id98.fasta \
 --ref /datanode03/liupf/software_lpf/sortmeRNA_db/silva-euk-18s-id95.fasta \
 --ref /datanode03/liupf/software_lpf/sortmeRNA_db/silva-euk-28s-id98.fasta \
 --fastx -a 15 -v --log \
 --reads sample.R1_trimmed.fq.gz \
 --reads sample.R2_trimmed.fq.gz \
 --aligned sample.align \
 --other sample.unalign \
 --paired_in --out2 --workdir ./sortmeRNA_temp

## megahit
conda activate megahit
megahit -1 sample.unalign_fwd.fq.gz -2 sample.unalign_rev.fq.gz -t 20 --out-prefix sample --out-dir ./megahit &> ./megahit.log
conda deactivate
```
---
## Prodigal, LucaProt, hmmsearch
```
# prodigal

source /apps/source/prodigal-2.6.3.sh
prodigal -p meta -q -m -i nt_seq.fa -a aa_seq.faa

## lucaprot
conda activate lucaprot
while read id
do
mkdir -p lucaprot/emb/${id}
cd ../LucaProt/src/
export CUDA_VISIBLE_DEVICES="0,1,2,3"
python predict_many_samples.py \
        --fasta_file /data/groups/lzu_public/home/liupf/zhujy/lucaprot/protein/${id}.faa  \
        --save_file /data/groups/lzu_public/home/liupf/zhujy/lucaprot/csv/${id}.csv \
        --emb_dir /data/groups/lzu_public/home/liupf/zhujy/lucaprot/emb/${id}   \
        --truncation_seq_length 4096  \
        --dataset_name rdrp_40_extend  \
        --dataset_type protein     \
        --task_type binary_class     \
        --model_type sefn     \
        --time_str 20230201140320   \
        --step 100000  \
        --threshold 0.5 \
        --print_per_number 10 \
        --gpu_id 0
cd /data/groups/lzu_public/home/liupf/zhujy
rm -r lucaprot/emb/${id}
done < id.list

## hmmsearch
source /datanode03/zhangxf/programs/mambaforge/bin/activate
conda activate hmmer

hmmsearch --cpu 60 -E 1 --tblout /path/to/output/dir/sample_id.txt /path/to/hmm/profile/all_RdRp.hmm /path/to/aa_seq/input_file.faa
```

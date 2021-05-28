Here describes workflow for genome assembly.

# 1. QC

## FastQC initial estimation
```
cd MGI2000/
for i in *.fq.gz; 
  do fastqc {i}; 
  done
```
## AfterQC automatic adapter trimming (because we don't have list of used adapters)

```
afterqc -1 v300039647_L03_100_1_fastqc.html -2 v300039647_L03_100_2_fastqc.html
afterqc -1 v300039647_L03_101_1_fastqc.html -2 v300039647_L03_101_2_fastqc.html
afterqc -1 v300039647_L03_102_1_fastqc.html -2 v300039647_L03_102_2_fastqc.html
afterqc -1 v300039647_L03_103_1_fastqc.html -2 v300039647_L03_103_2_fastqc.html
afterqc -1 v300039647_L03_104_1_fastqc.html -2 v300039647_L03_104_2_fastqc.html
```
## FastQC again

```
for i in *.fq.gz; 
  do fastqc {i}; 
  done
```
# 2. *De novo* genome assembly with HASLR

Nanopore reads were concatenated in single file for convinience of upload and then assmebly starts. Standard parameters were chosen for initial assembly. 
We specified genome assembly size (```-g```) to 3Gb and started it on 16 cores (```-t 16```).

```
haslr.py -t 16 -o ../haslr_res -g 3g -l ../reads_all.fastq -x nanopore \
  -s v300039647_L03_100_1.fq.gz v300039647_L03_100_2.fq.gz v300039647_L03_101_1.fq.gz \
  v300039647_L03_101_2.fq.gz v300039647_L03_102_1.fq.gz v300039647_L03_102_2.fq.gz \
  v300039647_L03_103_1.fq.gz v300039647_L03_103_2.fq.gz v300039647_L03_104_1.fq.gz v300039647_L03_104_2.fq.gz
```
**WARNING**: tool not produce final assembly due to unfixed bug or smth (see https://github.com/vpc-ccg/haslr/issues/18). But there is a 'solution':
- install HASLR via conda on your server where genome wiil be assembled;
- download source code of the tool to your PC, then ```make```;
- go to HASLR folder in your PC and copypaste **all** contetnt from this folder to conda env folder on your server. Overwrite all files.(Ypu may need FileZilla or other file manager to do it simple).

And after that tool should work. No other bugs and/or errors were met during the command working.
Unfortunately, no good assembly was produced -- only 12 Mb in size. Shame.

We than tried to reduce requirements to assembly quality(See tool documentation, direct link availbale in Workflow_intro):
- long read coverage (```--cov-lr```) was reduced by half (25-default, 12 was chosen);
- minimum length of alignment block (```--aln-block```) was reduced by half (500 default, 250 was chosen);
- minimum alignment similarity (```--aln-sim```) was reduced to 0.75 (default-0.85);
- minia kmer size (```--minia-kmer```) was reduced by half(default 49, 25 was chosen);
- minimum number of long read supporting each edge(```--edge-sup```) was reduced to 2(3 is default).

Resulting command is:

```
haslr.py -t 16 -o ../haslr_res_v2 -g 3g -l ../reads_all.fastq -x nanopore \
  -s v300039647_L03_100_1.fq.gz v300039647_L03_100_2.fq.gz v300039647_L03_101_1.fq.gz \
  v300039647_L03_101_2.fq.gz v300039647_L03_102_1.fq.gz v300039647_L03_102_2.fq.gz \
  v300039647_L03_103_1.fq.gz v300039647_L03_103_2.fq.gz v300039647_L03_104_1.fq.gz v300039647_L03_104_2.fq.gz
  --cov-lr 12 --aln-block 250 --aln-sim 0.75 --minia-kmer 25 --edge-sup 2
```
 Unfortunately, this didn't work good also. Assembly size was about 250Mb. Maybe something wrong with Nanopore data? We will back to this question in the next minute.
 
 # 3. Nanopore-only assembly
 As you know,we had WGS data made by Nanopore and MGI, and we can produce Nanopore-assembly also. So we decided to assemble genome only from Nanopore to be sure.
 We used 2 tools: Flye and wtdbg2,since both of them are fast and reasonably accurate.
 
 ## 3.1. wtdbg2
 ```
 wtdbg2 -i reads_all.fastq  -t 16 -x preset3 -g 3g -o assembly_HV6A
 ```
 ## 3.2. Flye
 ```
 flye --nano-raw reads_all.fastq --out-dir flye_res/ -g 3g -t 16
 ```
 We assessed qualities of both assemblies using QUAST.
 (insert QUAST res here)
 Most imporant thing we found is coverage -- only 4x, which is strangely low. Knowing this, we relaunched HASLR with changed (```--cov-lr```) to 4.
 
 # 4. *De novo* assembly with HASLR -- revisited
 ```
 haslr.py -t 16 -o ../haslr_res_v2 -g 3g -l ../reads_all.fastq -x nanopore \
  -s v300039647_L03_100_1.fq.gz v300039647_L03_100_2.fq.gz v300039647_L03_101_1.fq.gz \
  v300039647_L03_101_2.fq.gz v300039647_L03_102_1.fq.gz v300039647_L03_102_2.fq.gz \
  v300039647_L03_103_1.fq.gz v300039647_L03_103_2.fq.gz v300039647_L03_104_1.fq.gz v300039647_L03_104_2.fq.gz
  --cov-lr 4 --aln-block 250 --aln-sim 0.75 --minia-kmer 25 --edge-sup 2
```
 QUAST was used to assess genome quality.
 
 

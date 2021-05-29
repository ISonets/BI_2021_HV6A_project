Here describes workflow for genome assembly. Code and preliminary results presented here are at 22/05/2021.
This part was made by me, Ignat Sonets. 
Many thanks to Yuri Barbitov and Yulia Yakovleva for provided help and useful advices!

# 1. QC

## 1.1. FastQC initial estimation
FastQC was installed through conda.
```
cd MGI2000/
for i in *.fq.gz; 
  do fastqc {i}; 
  done
```
## 1.2. AfterQC automatic adapter trimming (because we don't have list of used adapters)
AfterQC was installed through conda.

```
afterqc -1 v300039647_L03_100_1_fastqc.html -2 v300039647_L03_100_2_fastqc.html
afterqc -1 v300039647_L03_101_1_fastqc.html -2 v300039647_L03_101_2_fastqc.html
afterqc -1 v300039647_L03_102_1_fastqc.html -2 v300039647_L03_102_2_fastqc.html
afterqc -1 v300039647_L03_103_1_fastqc.html -2 v300039647_L03_103_2_fastqc.html
afterqc -1 v300039647_L03_104_1_fastqc.html -2 v300039647_L03_104_2_fastqc.html
```
## 1.3. FastQC again

```
for i in *.fq.gz; 
  do fastqc {i}; 
  done
```
# 2. *De novo* genome assembly with HASLR

Nanopore reads were concatenated in single file for convinience of upload and then assmebly starts (```reads_all.fastq```). Standard parameters were chosen for initial assembly. 
I specified genome assembly size (```-g```) to 3Gb and started it on 16 cores (```-t 16```).

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

I than tried to reduce requirements to assembly quality(See tool documentation, direct link availbale in Workflow_intro):
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
Unfortunately, this didn't work good also. Assembly size was about 250Mb. Maybe something wrong with Nanopore data? I will back to this question in the next minute.
 
 # 3. Nanopore-only assembly
As you know,we had WGS data made by Nanopore and MGI, and I can produce Nanopore-assembly also. So I decided to assemble genome only from Nanopore to be sure.
I used 2 tools: Flye and wtdbg2,since both of them are fast and reasonably accurate.
I launched both tolls with standard parameters.
 
## 3.1. wtdbg2
Tool was installed through conda.
I specified threads number (```-t 16```) and genome size (```-g 3g```) (3Gb). Log file is available at logs/ folder. Total size of the assembly is 265Mb. Bad.
```
wtdbg2 -i reads_all.fastq  -t 16 -x preset3 -g 3g -o assembly_HV6A
```
## 3.2. Flye
Tool was installed through conda.
I specified threads number (```-t 16```) and genome size (```-g 3g```) (3Gb). Log file is available at logs/ folder. Total size of the assembly is 180Mb. Even worse, and far from perfect.
```
flye --nano-raw reads_all.fastq --out-dir flye_res/ -g 3g -t 16
```
I assessed qualities of both assemblies using QUAST. QUAST results are in QUAST_res/ folder.
Most imporant thing I found is coverage -- only 4x, which is strangely low. Knowing this, I relaunched HASLR with changed (```--cov-lr```) to 4.
 
# 4. *De novo* assembly with HASLR -- revisited
```
haslr.py -t 16 -o ../haslr_res_v2 -g 3g -l ../reads_all.fastq -x nanopore \
 -s v300039647_L03_100_1.fq.gz v300039647_L03_100_2.fq.gz v300039647_L03_101_1.fq.gz \
 v300039647_L03_101_2.fq.gz v300039647_L03_102_1.fq.gz v300039647_L03_102_2.fq.gz \
 v300039647_L03_103_1.fq.gz v300039647_L03_103_2.fq.gz v300039647_L03_104_1.fq.gz v300039647_L03_104_2.fq.gz
 --cov-lr 4 --aln-block 250 --aln-sim 0.75 --minia-kmer 25 --edge-sup 2
```
QUAST was used to assess genome quality. QUAST results are in QUAST_res/ folder. Size of the assembly is 752Mb. Far from perfect. But may be a good start for improved assemblies.
 
# 5. *De novo* genome assembly with WENGAN
Tool was installed via building from source. Not all components were built. ABYSS and DiscovarDeNovo builds failed, I tried to fix it, but also failed. Fortunately, Minia3 built itself correctly.
I changed some of the parameters to lauch WENGAN as intended:
- assembler mode(```-a```) set up to ```M```(Minia3 assembler,fastest mode with reasonable RAM usage);
- 16 threads(```-t 16```) was selected;
- genome size(```-g 3000```) was set up.
 
**WARNING**: LIGER scaffolding tool which is component of WENGAN was not run properly in automatic mode. No eror messages occured, but in results folder we didn't found any final results. We decided to run LIGER directly (by copying last command that occurs in log)
```
perl ../wengan/wengan.pl -x ontraw -a M -s v300039647_L03_100_1.fq.gz,v300039647_L03_100_2.fq.gz,\
v300039647_L03_101_1.fq.gz,v300039647_L03_101_2.fq.gz,\ 
v300039647_L03_102_1.fq.gz,v300039647_L03_102_2.fq.gz,v300039647_L03_103_1.fq.gz,\
v300039647_L03_103_2.fq.gz,v300039647_L03_104_1.fq.gz,v300039647_L03_104_2.fq.gz -l reads_all.fastq.gz -p asm1 -t 16 -g 3000
```
```
../wengan/bin/liger  --mlp 1000 --nlm 3 --mcl 300 --mcs 1000 --mcr 1000 --lme 10000 -t 16 -c asm1.MBC7.msplit.fa \
-l longreads.asm11.fa asm1.MBC7.msplit.cov.txt -p asm1 -s asm1.sams.txt>asm1.liger.err>asm1.liger.log
```
And some results occured. Assembly quality was estimated with QUAST. QUAST results are in QUAST_res/ folder.
Unfortunately results were...1.5Mb. I have no idea how this could exist. I suggested that LIGER parameters should also be changed, but **no** documentaton for LIGER module is provided, and random changing them...I scrapped this assembler tool results and moved along.

# 6. MGI-only assembly.
De novo assembly results looked terrible, and I decided to scrap results and try to assemble MGI-only assembly.
2 tools were selected for an assembly: ABYSS and Minia.

## 6.1. ABYSS
Installation of this tool was a big pain. If you don't have root on server, so sparsehash library(at least this 1 library) won't install and you will have a bunch of errors while compiling and building from source. Use conda package instead, but install it to the new clean environment,because I entered some dependency conflicts and I had no desire to solve them one by one.
I concatenated all R1 and R2 reads into 2 big files using ```cat```:

```
gunzip *.fq.gz
cat *_1.fq > reads_R1_MGI.fastq
cat *_2.fq > reads_R2_MGI.fastq
```
Usage is simple. I specified mode for paired-end libraries (```abyss-pe```), specified number of threads (16) (```np=16```) and k-mer size (```k=96```). Total size is about 210 Mb. Weird...
Log file is available at logs/ folder. 

```
 abyss-pe name=hs_HV6A k=96 in='reads_1_MGI.fq reads_2_MGI.fq' np=16
```
QUAST was used to estimate assembly quality. QUAST results are in QUAST_res/ folder.

# 6.2. Minia

Tool was installed with conda. No problem. Assuming fo FastQC report (which showed good quality), I decided to launch the tool with standard parameters.
Log file is available at logs/ folder. QUAST results are in QUAST_res/ folder. Total size of the assembly is about 3.1 Gb.
```
minia -in reads_1_MGI.fq,reads_2_MGI.fq -out minia_asm.fa
```
Minia showed the best resulting assembly across all draft assemblies. But it is only partial success. After some consulting , I decided to adopt custom pipeline.

# 7. Custom pipeline

Fig.1 illustates custom pipeline prosed for task completion.

![Fig.1. Pipeline proposal](/custom_pipeline_proposal.png)

Fig.1. Pipeline proposal

   - first assemble Nanopore reads using Flye (done previosly);
   - polish them with Minimap2 and Racon (we are here at 21/05/21);
   - align MGI reads to the draft assembly with BWA and polish gaps with Pilon;
   - reassemble contigs to scaffold using Flye.
Unfortunately, Racon polisher refused to work as intended, and I have no idea why. It said smth about malformed header and errors in generated .sam files,(log file was deleted by my mistake) although they looked completely normal. It seems that we should do it again, with clean env and maybe realign raw reads and assembly again, and maybe try another aligner (not Minimap2).

# 8. Taxonomic analysis.

Maybe my reads are totally contaminated, and I can't produce good alignments and therefore good contigs and scaffolds? I decided to use Kaiju to verify taxonomic content.
Tool was installed via conda. As a reference database RefSeq Genomes database were used and downloaded. To put in simple, there are some contamination, most of them are viral and bacterial, bus some fungi also occur in raw data. Unfortunately, I ran out of time and don't check all reads, but I assume that because all data are taken from 1 person and processed simultaneously, results would be relevant. See Fig.2 (Nanopore data taxonomic profile), Fig.3 (MGI data taxonomic profile) here and in Kaiju_results/ folder (as also Table 1 (taxonomic profile summary for Nanopore reads) and Table 2 (taxonomic profile summary for MGI reads)) for more details.
Fig.2. Kaiju result for nanopore reads]
![Fig.2. Kaiju result for nanopore reads](/Kaiju_results/Fig_2_nanopore_taxa_result.png)


Fig.3. Kaiju result for MGI reads
![Fig.3. Kaiju result for MGI reads](/Kaiju_results/Fig_3_MGI_taxa_result.png)

In future I plan to filter out bacterial reads and preserve viral reads,since we seacrhing for viral genome.

# 9. Future plans and things to discuss:

- Nanopore resequencing?
- moving to new cluster with more powerful hardware?
- filter out bacterial contamination?
- more consulting?

If you have any advices/tips and hints/recommendations and even critique, feel free to email me (ignatsonets@gmail.com).
Thank you for your attention!


 

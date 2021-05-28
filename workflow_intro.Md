Workflow for HV6A project

0. Overview of data and hardware used in this project

DATA:
1) Oxford Nanopore raw data after basecalling (about 10.2 Gb) if .FASTQ format
2) MGI raw data (paired-end sequencing) in .FASTQ format(about 20 Gb)
There is data form whole-genome sequencing of 1 patient (father of the child mentioned in Background section of README).
Sequencedmaterial was taken from spermatozoids. 
For more info about DNA isolation, library preparation and sequencing parameters please contact Olga Goleva, PhD(golev.ao@mail.ru) 

HARDWARE:
Cluster of our lab: 3 Tb availbale space, 32 CPU, 190Gb of RAM

1. TOOLS used in this study:
  1. QC-- FastQC (https://github.com/s-andrews/FastQC). See FastQC reports after trimming in specified folder.
  2. For MGI data adapter trimming (as also bad quality base trimming) -- AfterQC (https://github.com/OpenGene/AfterQC)
  NB! QC was not performed for nanopore data. Why? As I know, telomeres and sub-telomeres regions are bad at sequencing, and at the same time integration site locates approximately in these regions. 
  So, to not lose taget sequences,we had to left them as is.
  3. Genome de novo assembly tools:
    3.1. de novo hybrid assembly:
        -HASLR (https://github.com/vpc-ccg/haslr)
        -WENGAN (https://github.com/adigenova/wengan)
        We chose these tools mainly due to their low requirements to CPU in comparison with MASURCA or other time/power-consuming tools.
        However, these tools are currently in early stages development, and this might be the reason for our contoversial results.
    3.2. Nanopore-only genome assembly -- Flye (https://github.com/fenderglass/Flye) and wtdbg2 (
    3.3. Illumina-only genome assembly -- Minia (https://github.com/GATB/minia)
    Our main limitation was limited possibilities of available hardware. Assembly of Nanopore/MGI-on;y reads was performed as an alternative assembly that migth be useful on next steps of our project.
    3.4. Custom pipeline(see Fig.1 in repository).
        Other ready-made pipelines were not successful, so we decided to make custom pipeline. Its main steps are:
        - first assemble Nanopore reads using Flye;
        - polish them with Minimap2 and Racon;
        - align MGI reads to the draft assembly with BWA and polish gaps with Pilon;
        - reassemble contigs to scaffold using Flye.
        On 30/05/21, this pipeline is WIP and may change.
    3.5. Assessing quality of assemblies: QUAST (https://github.com/ablab/quast)
    3.6. Assessing contamination of reads: Kaiju (https://github.com/bioinformatics-centre/kaiju)
    3.7. For searching of integration site: Minimap2 (https://github.com/lh3/minimap2)  
 

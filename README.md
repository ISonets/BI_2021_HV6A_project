# BI 2021 HV6A_project
BI 2021 Spring Project

Students:
1) Ignat Sonets;
2) Adelia Popova

This repo contains only part done by Ignat Sonets, Adelia has her own repository to store data and results. Project is now in WIP and work will be continued after 15/06/2021.

# PROJECT BACKGROUND:

It has been shown that human betaherpesvirus 6 (HHV-6) can integrate into the telomeres of the host cell chromosomes.
 
The chromosomally integrated form of the virus (chiHVH-6) occurs in the human population up to 1.5% of cases and can be inherited from one or both parents to children. The sites of integration are not clearly defined, although it is assumed that viral integration occurs predominantly in certain chromosomes. There is evidence of the possibility of reactivation of HHV-6 integrated into the human chromosome during life in infected individuals. Confirmation of the reactivation of chiHVH-6 with the development of an active infectious process will be an important factor in deciding whether to prescribe etiotropic therapy.

A clinical case of constant isolation of HHV-6A DNA in constant concentrations from the blood and scraping of the oropharynx in a 6-year-old patient of the clinic, hospitalized for rhinopharyngitis, as well as in the examined members of his family (older brother and father) who did not complain on the state of health. In addition, HHV-6A DNA was also detected in the secret of the gonads of the father of the hospitalized patient. A possible chromosomal integration of the virus into human DNA, inherited through the paternal line to children, does not cause a pathological process, which requires further in-depth study.

Thus, the study is devoted to the localization of the site of HHV-6A integration into the human chromosome. Full genome sequencing of DNA obtained from biological materials of the father (blood, secretions of the testicles) was performed using the following technologies: Illulmina, BGI, Oxford Nanopore Technology.

The aim of this work is a hybrid assembly of the human genome, presumably containing integrated HHV-6A. The results may be relevant for further clarifying studies (Sanger sequencing, FISH hybridization)

See Workflow_genome_assembly.Md for workflow of this project.

# TASKS:

1. Analyze the available algorithms for performing de novo hybrid assembly of the human genome.
2. Perform hybrid de novo assembly with at least two alternative algorithms. Compare the build quality in terms of NG50, NGA50.
3. Predict assembly errors using the GRCh38 reference genome.
4. Search for the site of the HV6A insertion.

# Students & their tasks:

1) Ignat Sonets -- hybrid de novo genome assembly
2) Adelia Popova -- HHV6A virus integration site search (see separate repository)

# REPOSITORY STRUCTURE:
1) Workflow_genome_assembly.Md -- main document, where you can find list of used tools, hardware desciption, wokflow, struggels during the project and future plans;
2) FastQC_reports -- reports for all MGI reads data after using AfterQC tool;
3) Kaiju results -- results for estimation taxonomic composition. To date only 2 of the files were checked,but we suggest that other files being taken from 1 source also had same taxonomic profile;
4) QUAST_res -- statistics for almost all assemblies made for this project;
5) assembly_logs -- log files for almost all assemblies, this files could be useful in future work.

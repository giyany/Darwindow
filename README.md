# Darwindow

Darwindow is a small tool, consisting of a Unix and a R script, to calculate and plot population-genetic estimates on a sliding-window basis, using as input a gVCF file  (i.e., a vcf-file containing information on both monomorphic and polymorphic sites for the entire genome). This input gVCF can be generated with the enclosed BAM2VCF script, but of course, users are free to generate this gVCF file with the software/pipeline of their own preference.    

Darwindow is particularly useful to perform run of homozygosity (ROH) analyses. The advantage of Darwindow compared to other ROH-detection software is that Darwindow allows the user to evaluate the accuracy of the ROH calls through visual examination of heterozygosity levels in regions marked as ROH (grey rectangles in plots with prefix 'He_withROH_linechart', generated by the runindscaffold function).

Note that it only makes sense to perform ROH-analyses if you mapped your data against a high-quality genome assembly (high scaffold N50-value), preferably chromosome-level. For instance, if the largest scaffolds are only 10Mb in length, it is not possible to determine the presence of ROHs over 10Mb length. Even smaller ROHs (say 5Mb) will be difficult to detect, because chances are such a ROH is distributed over different scaffolds.      

![alt text](https://github.com/mennodejong1986/Darwindow/blob/main/ROH_regions.png)
***Figure 1. Darwindow allows visual examination of regions marked as run of homozygosity.*** *Variation of heterozygosity levels (He) across a randomly chosen chromosome, for a random subset of brown bear samples, depicting He-levels of non-overlapping 20 kb windows, with grey segments highlighting the genomic regions which are marked as run of homozygosity (ROH) by Darwindow. Values along the y-axis represent sample and chromosome-specific FROH-values (proportion of chromosome marked as ROH).*


## How to run 'VCF_darwindow.sh'

In order to run Darwindow, follow the instructions in the top section of the VCF_darwindow.sh script.
This will generate file(s) containing window-based counts. 

The main output file will start with the prefix 'mywindowhe' and contains the columns:
'contig startbp endbp totalbp nmiss_1 nsites_1 nhet_1 nhomo_1 nmiss_2 nsites_2 nhet_2 nhomo_2 nmiss_3 nsites_3 nhet_3 nhomo_3 etc'

The columns 'nmiss_1', 'nsites_1', 'nhet_1' and 'nhomo_1' give for each window the number of missing sites, the number of non-missing sites, the number of heterozygous sites and the number of alternative homozygous sites observed for individual 1 (first individual in the vcf-file). The columns 'nmiss_2', 'nsites_2', 'nhet_2' and 'nhomo_2 give these counts for individual 2, and so on. 


## How to next run 'VCF_windowhe_plotinR.txt' 

The main output file can be further analysed and plotted in R using the script 'VCF_windowhe_plotinR.txt'.
To do so for the example dataset (135 bears, 3 chromosomes), execute in R the following commands:

**First load all required functions:**

*source("VCF_darwindow.plotinR.txt")*

*if(!"zoo"%in%rownames(installed.packages())){install.packages("zoo")}*

*if(!"graphics"%in%rownames(installed.packages())){install.packages("graphics")}*

*library("zoo")*

*library("graphics")*

**Next define the settings:**

Window size in bp (should correspond to window size used when running VCF_darwindow.sh script):

*window_size	<- 20000*		  

Minimum number of adjacent windows to be considered as a ROH (for example: if n_windows is set 10, and window size is 20000, then reported ROHs are at minimum 200Kb):

*nr_windows	<- 10*			    

Maximum amount of missing data per window:

*miss_max	  <- 0.8*         


**Import data:**

*mywd		<- "C:/path/to/directory/"*

*setwd(mywd)*

*getwindowdata(maxmiss=0.8,suffix="20000.brownbears.three_scaffolds.txt",vcfsamples="myvcfsamples.txt",samplefile="popfile.txt",annotated=FALSE,indlevel=TRUE,poplevel=FALSE,mydir=mywd)*

**Optionally define population order (to be used in some of the output plots):**

*mypoporder	<- c("MiddleEast","Himalaya","Europe","SouthScand","MidScand","NorthScand","Baltic","Ural","CentreRus2","CentreRus","Yakutia","Amur","Hokkaido","Sakhalin","Magadan","Kamtchatka","Aleutian","Kodiak","Alaska","ABCa","ABCbc","ABCcoast2","Westcoast","ABCcoast1","HudsonBay","polar","Black")* 

*reorder_pop(poporder=mypoporder)*

**Calculate genome-wide heterozygosity:**

*calcwindowhe(maxmiss=miss_max)*

*calcregionhe(maxmiss=miss_max,nwindows=nr_windows)*

**Plot heterozygosity:**

*popboxplot(export="pdf",mywidth=0.5)*

*indhisto(export="pdf",plotname="He_histo_region",inputdf=dwd$regionhedf,missdf=NULL,windowsize=window_size,nwindows=nr_windows,mybreaks=seq(-0.01,5,0.005),xmax=0.55,ymax=15,legendcex=1)*

*indbarplot(export="pdf",mywidth=0.5)*

*indboxplot(export="pdf",inputdf=dwd$hedf,plotname="Genomewide_windowHe",ylabel="Heterozygosity (%)",yline=3.25,samplesize=500,maxmiss=miss_max,ymax=0.95,mywidth=0.5)*

![alt text](https://github.com/mennodejong1986/Darwindow/blob/main/Heterozygosity_plots.png)
***Figure 2. Genome-wide heterozygosity.*** *Left: Boxplot with sample-specific genome-wide heterozygosity scores, grouped by population. Right: comparison between Darwindow and bcftools heterozygosity estimates.The average genome-wide heterozygosity estimates produced by Darwindow are near-identical to the estimates obtained with 'bcftools stats -s -', assuming that indels have been removed from the input vcf-file using the command 'zgrep -v "INDEL" data.vcf.gz | gzip > data.noindels.vcf.gz'. (This step is included in Darwindow pipeline).* 

**Detect runs of homozygosity:**

First you need to define the maximum heterozygosity threshold for a ROH. This depends on the mean genome-wide heterozygosity. For example, if the mean heterozygosity is 0.25%, you might set the threshold to 0.05. If the mean heterozygosity is 0.10%, you might want to set the threshold to 0.02%.

One option is to specify a universal threshold value for all individuals:

*hethres_vec	<- 0.05*	

The second option is to specify a threshold per individual:

*hethres_vec	<- ifelse(dwd$ind$pop=="polar",0.001,0.05)*

Once you defined the threshold, run the following functions to detect ROHs:

*findroh(silent=TRUE,hethreshold=hethres_vec,min_rle_length=1,windowsize=window_size,nwindows=nr_windows)*

*getrohlengths(windowsize=window_size,nrwindows=nr_windows)*

*getrohbin()*

**Plot runs of homozygosity:**

*runindscaffold(height_unit=1,do_export=TRUE,input_df1=dwd$hedf,input_df2=dwd$frohdf,plot_label="He_withROH",add_roh=TRUE,add_he=TRUE,add_dxy=FALSE,max_miss=miss_max,n_windows=nr_windows,min_rle_len=1,window_size=window_size,line_width=0.1,plot_width=14)*

Visually examine the output line charts (see Figure 1). 
In case your scaffolds are relatively small (e.g., smaller than 50 Mb), it might proof helpful (for visualisation purposes) to narrow the width of the plot using the plot_width flag.
Do regions marked as run of homozygosity (grey areas) indeed have low heterozygosity?
If not, try different settings (i.e.: use different values for he_thres_vec and/or nr_windows).
If yes, then you can proceed and create the final plots with ROH summary statistics.

**Plot ROH summary statistics:**

*popboxplot(export="pdf",ymax=NULL,indscore="froh",plotname="Genomewide_froh",ylabel="F_roh",mywidth=0.5)*

*indscatter(export="pdf",plotname="Lroh_vs_Nroh",xscore="nroh",yscore="lroh",xlabel="Number of ROHs",ylabel="Total ROH length (Mb)",legendpos="bottomright",legendcex=1,yline=5.5)*

*indscatter(export="pdf",plotname="Froh_vs_He",xscore="regionhe",yscore="froh",xlabel="Genome wide He",ylabel="F (ROH-content)",legendpos="topright",legendcex=0.85,yline=5.75)*

*indscatter(export="pdf",plotname="Froh_mean_vs_sd",xscore="froh",yscore="froh_sd_scaffold",xlabel="F-roh mean",ylabel="F-roh sd (across chromosomes)",addlegend=FALSE,yline=5.75)*

The most informative ROH summary plot is arguably the stacked barplot:

*rohbarplot(inputdf=dwd$frohbindf,ylabel="F-roh",plotname="ROHf_barplot",export="pdf",yline=3,mywidth=0.2,legendcex=1.75,addlegend=TRUE,mycolours=NULL,ypopcol=0.78,legx=30,legy=0.725)*

*rohbarplot(inputdf=dwd$nrohbindf,ylabel="# ROHs",plotname="ROHn_barplot",export="pdf",yline=4.5,mywidth=0.2,legendcex=1.75,addlegend=TRUE,mycolours=NULL,ypopcol=1850,legx=20,legy=1700)*

![alt text](https://github.com/mennodejong1986/Darwindow/blob/main/ROH_barplot.png)
***Figure 3. Stacked barplots.*** *Sample specific ROH-distributions. The colour bar above the stacked barplot indicates population assignment.*

If you want to use the plot for scientific posters, you could vary the background colour:

*rohbarplot(inputdf=dwd$frohbindf,ylabel="F-roh",plotname="ROHf_barplot",export="pdf",yline=3,mywidth=0.2,legendcex=1.75,addlegend=TRUE,mycolours=NULL,ypopcol=0.78,legx=30,legy=0.725,mybg="lightblue4",axiscol="grey80")*

## How to cite Darwindow

If you use Darwindow for your own research, please cite:

*De Jong et al., 2023, Range-wide whole-genome resequencing of the brown bear reveals drivers of intraspecies divergence. Commun Biol 6, 153. https://doi.org/10.1038/s42003-023-04514-w*



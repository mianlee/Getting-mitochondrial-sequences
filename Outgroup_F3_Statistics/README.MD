F-statistics (not Wright’s fixation indexes, Fst), a simple and intuitive approach to decipher more detailed and complex questions about our history, has quickly gained in popularity. It is first time introduced by [Reich et al. (2009)](https://www.nature.com/articles/nature08365) and summarized in [Patterson et al. (2012)](https://www.genetics.org/content/192/3/1065). In that framework, inference is based on “shared genetic drift” between sets of populations, under the premise that shared drift implies a
shared evolutionary history. -- [Peter,BM. (2016)](https://www.genetics.org/content/202/4/1485.abstract)


The “drift” is defined as the frequency change of an allele along a graph edge (hence drift between two populations A and B is a function of the difference in the allele frequency of polymorphisms in A and B), [Patterson et al. (2012)](https://www.genetics.org/content/192/3/1065).


F3 statistics are used for two purposes:

1. F3 Admixture Analysis, to look for significant evidence of admixture, as a test whether a target population (C) is admixed between two source populations (A and B). Theory shows that given allele frequencies at a given SNP for three populations, A, B and C, the F3 statistic [(C-A)(C-B)] is only negative if a target population (C) was the product of admixture between two source populations A and B (or their ancestors). [AAAGs_2018_Session8](https://github.com/Sagui-omics/AAAGs_2018/tree/master/Session8_Admixture_Introgression), [GAWorkshop](https://gaworkshop.readthedocs.io/en/latest/contents/06_f3/f3.html).

2. Outgroup F3 statistics, to measure shared genetic drift between two test populations (A and B) from an outgroup (C).


F3 statistics are in both cases defined as the product of allele frequency differences between population C to A and B, respectively:

```F3(C;A,B) = E[(c−a)(c−b)].```

Here, E[ ] denotes the average over all genotyped sites, and a, b and c denote the allele frequency in the three populations. Outgroup F3 statistics measure the amount of shared genetic drift between two populations from a common ancestor. In a phylogenetic tree connecting A, B and C, Outgroup F3 statistics measure the common branch length from the outgroup, here indicated in red:

![](Images/f3-tree.png)

[GAWorkshop](https://gaworkshop.readthedocs.io/en/latest/contents/06_f3/f3.html)


### Exctract all 92 ancient individuals and Yoruba population as outgroup

Files:

[ancient_Yoruba_list.txt](ancient_Yoruba_list.txt), ```data.bed```, ```data.bim```, ```data.fam```.


```
plink --bfile data --keep ancient_Yoruba_list.txt --make-bed --out ancient_92_Yoruba

# Log file
PLINK v1.90b4 64-bit (20 Mar 2017)             www.cog-genomics.org/plink/1.9/
(C) 2005-2017 Shaun Purcell, Christopher Chang   GNU General Public License v3
Logging to ancient_92_Yoruba.log.
Options in effect:
  --bfile data
  --keep ancient_Yoruba_list.txt
  --make-bed
  --out ancient_92_Yoruba

8703 MB RAM detected; reserving 4351 MB for main workspace.
354212 variants loaded from .bim file.
2071 people (1307 males, 764 females) loaded from .fam.
2071 phenotype values loaded from .fam.
--keep: 162 people remaining.
Using 1 thread (no multithreaded calculations invoked).
Before main variant filters, 162 founders and 0 nonfounders present.
Calculating allele frequencies... done.
Total genotyping rate in remaining samples is 0.773673.
354212 variants and 162 people pass filters and QC.
Among remaining phenotypes, 0 are cases and 162 are controls.
--make-bed to ancient_92_Yoruba.bed + ancient_92_Yoruba.bim +
ancient_92_Yoruba.fam ... done.
```

```
nano par.packedped.PACKEDANCESTRYMAP 


genotypename: ancient_92_Yoruba.bed
snpname: ancient_92_Yoruba.bim
indivname: ancient_92_Yoruba.fam
outputformat: PACKEDANCESTRYMAP
genotypeoutname: ancient_92_Yoruba.geno
snpoutname: ancient_92_Yoruba.snp
indivoutname: ancient_92_Yoruba.ind
```

Fix the ***.ind** file:

```
awk -F ":" '{print $1 "\t" $2}' ancient_92_Yoruba.ind | awk '{print $1":"$2 "\t" $3 "\t" $1}' > fixed.ind

mv fixed.ind ancient_92_Yoruba.ind
```
[ancient_92_Yoruba.ind](ancient_92_Yoruba.ind)


For computing F3 statistics including error bars, we will use the ```qp3Pop``` program from [Admixtools](https://github.com/DReichLab/AdmixTools).

One more ingredient that you need is a list of population triples. This should be a file with three population names in each row, separated by space or tab. So I am going to make this population file using [pop.list.txt](pop.list.txt).

```
cat pop.list.txt | while read line
do 
grep -v "$line" pop.list.txt | awk -v s=$line '{print s "\t" $1 "\t" "Yoruba"}' >> ancientpopfile.txt
done

```
Here, the ```while read ``` command loops through all rows (eahc population) in [pop.list.txt](pop.list.txt), ```line``` is a variable name (you can you other names, like sample, n...). First column ```s``` is the population A, the second column are other populations (B1, B2, B3...), and “Yoruba” as an outgroup in last position. (assigned as variable s in ```awk``` through a command line option ```-v s=$line```). See the [ancientpopfile.txt](ancientpopfile.txt).


In the Haak et al 2015, the authors used **Dinka** as an outgroup. However, the dataset they provided doesn't have Dinka population, so I use Yoruba instead. The authors mentioned in the paper that "results with other African outgroups such as Yoruba or Ju_hoan_North (San) are qualitatively similar".


Similar to the ```convertf``` and the ```smartpca``` programs we have already used, ```qp3Pop``` requires a parameter file as input. 
In my case, it looks like this:

```
nano par.f3.ancient


genotypename: ancient_92_Yoruba.geno
snpname: ancient_92_Yoruba.snp
indivname: ancient_92_Yoruba.ind
popfilename: ancientpopfile.txt
```

ADMIXTOOLS version 5.1  9/18/18 (for Linux and Mac)

```
/PATH/TO/AdmixTools_master/bin/qp3Pop -p par.f3.ancient > f3.results

```
[f3.result](f3.result)


As you see, we don’t want columns 1 and comment rows. You can use ```awk``` to keep only columns 2, 3, 4, 5, 6, 7, 8:

```
grep "result:" f3.result | awk '{print$2 "\t" $3 "\t" $4 "\t" $5"\t" $6 "\t" $7 "\t" $8}' > f3.ancient.results
```
[f3.ancient.results](f3.ancient.results)

### Plot in R

```
f3data <- read.table("f3.ancient.results", col.names=c("PopA", "PopB", "Outgroup", "F3", "StdErr", "Z", "SNPs"))

attach(f3data)

# Extract certain comparisions 

Loschbour <- f3data[PopA == "Loschbour", ]   # Loschbour vs other populations

Odered_Loschbour <- Loschbour[order(Loschbour$F3),]  # $F3 means by f3 values



# Plot f3 values with standard error
library(ggplot2)

ggplot(Odered_Loschbour, aes(y = F3, x = factor(Odered_Loschbour$PopB, levels = Odered_Loschbour$PopB))) +
  geom_errorbar(aes(ymin = F3-3*StdErr, ymax = F3+3*StdErr), width = 0.0, size = 0.5, color = "grey") +
  geom_point(size = 2) +
  coord_flip() +
  theme_bw() +
  theme(panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        axis.line.x = element_line(colour = "black"),
        axis.title = element_blank(),
        axis.text.y = element_blank(),
        axis.text.x = element_text(size =12, margin = margin (t = 9) ),
        axis.ticks.y = element_blank(),
        axis.ticks.length = unit(0.2, "cm") ) +
  geom_text(aes(label=PopB),position = position_nudge(y = -0.024))



library(ggrepel) #tool for labelling in ggplot

```
![](Images/Loschbour.png)

Refined Plot

```
f3data <- read.table("f3.ancient.results", col.names=c("PopA", "PopB", "Outgroup", "F3", "StdErr", "Z", "SNPs"))

attach(f3data)

# Extract certain comparisions 

Loschbour <- f3data[PopA == "Loschbour", ]   # Loschbour vs other populations

Odered_Loschbour <- Loschbour[order(Loschbour$F3),]  # $F3 means by f3 values


ggplot(Odered_Loschbour, aes(y = F3, x = factor(Odered_Loschbour$PopB, levels = Odered_Loschbour$PopB))) +
  geom_errorbar(aes(ymin = F3-3*StdErr, ymax = F3+3*StdErr), width = 0.0, size = 0.5, color = "grey") +
  geom_point(size = 2) +
  ylim(0.11, 0.225)+
  coord_flip() +
  theme_bw() +
  theme(panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        axis.line.x = element_line(colour = "black"),
        axis.title = element_blank(),
        axis.text.y = element_blank(),
        axis.text.x = element_text(size =12, margin = margin (t = 9) ),
        axis.ticks.y = element_blank(),
        axis.ticks.length = unit(0.2, "cm")) +
  geom_text(aes(y = F3-3*StdErr,label=PopB), hjust =1.1)
```
![](Images/Loschbour_refined.png)


**Reference**:

1. Reich, D., Thangaraj, K., Patterson, N., Price, A. L., & Singh, L. (2009). Reconstructing Indian population history. Nature, 461(7263), 489.

2. Patterson, N., Moorjani, P., Luo, Y., Mallick, S., Rohland, N., Zhan, Y., ... & Reich, D. (2012). Ancient admixture in human history. Genetics, 192(3), 1065-1093.

3. Peter, B. M. (2016). Admixture, population structure, and F-statistics. Genetics, 202(4), 1485-1501.

4. Haak, W., Lazaridis, I., Patterson, N., Rohland, N., Mallick, S., Llamas, B., ... & Fu, Q. (2015). Massive migration from the steppe was a source for Indo-European languages in Europe. Nature, 522(7555), 207.

Data for gene expression example 
================================

Three files with information from GTEx project were downloaded from [GTEx portal](https://gtexportal.org). Login needed to download from `Datasets>Download`

1. __RNA-Seq Data, median TPM by tissue__  
Median expression for 56202 genes in 53 tissues `GTEx_Analysis_2016-01-15_v7_RNASeQCv1.1.8_gene_median_tpm.gct.gz`
2. __Annotations, de-identified sample annotations__  
For more detailed classification of tissues `GTEx_v7_Annotations_SampleAttributesDS.txt`.
3. __Reference, Gene models__  
To have more details about the genes `gencode.v19.genes.v7.patched_contigs.gtf`.


Data cleaning
-------------

Simple list with tissue type `tissue_info.txt`
```{bash}
echo -e "tissue_type\ttissue" >tissue_info.txt
cut -f6-7 GTEx_v7_Annotations_SampleAttributesDS.txt | tail -n+2 | sort | uniq | tr " " "." | tr "-" "." | tr "(" "." | tr ")" "." >>tissue_info.txt
# Remove Bone marrow (not present in the expression table)
grep -vE "^Bone.Marrow" tissue_info.txt > tissue_info_tmp.txt
mv tissue_info_tmp.txt tissue_info.txt
```

Simple list of genes, positions, type... `gene_info.txt`
```{bash}
echo -e "Chromosome\tSource\tStart\tEnd\tStrand\tID\tType\tStatus\tName" > gene_info.txt
grep -vE "^##"  gencode.v19.genes.v7.patched_contigs.gtf | awk '$3=="gene"' | tr ";" "\t" | tr " " "\t" | tr -s "\t" | cut -f1,2,4,5,7,10,14,16,18 >> gene_info.txt
```

Reshape median gene expression per tissue, add the other info and subset 8 tissues for the exercise
```{r}
expression_matrix<-read.table("GTEx_Analysis_2016-01-15_v7_RNASeQCv1.1.8_gene_median_tpm.gct.gz",skip = 2, header = TRUE, sep = "\t", stringsAsFactors = FALSE)
library(reshape2)
expression_data<-melt(expression_matrix, id.vars = c("gene_id", "Description"), value.name = "median_expression", variable.name = "tissue")

# Add info
tissue_info<-read.table("tissue_info.txt", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
expression_data_2<-merge(expression_data, tissue_info)
gene_info<-read.table("gene_info.txt", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
expression_data_3<-merge(expression_data_2,gene_info,by.x = "gene_id",by.y = "ID")
colnames(expression_data_3)<-c("gene_id","gene_name","tissue","median_expression", "tissue_type", "gene_chr", "gene_annotation", "gene_start", "gene_end", "gene_strand", "gene_type", "gene_status")

write.table(expression_data_3, file = "expression_data_complete.txt", sep = "\t", row.names = FALSE, quote = FALSE)

# Subset 8 tissues
selected_tissues <- c("Brain...Cortex", "Liver", "Testis", "Muscle...Skeletal", "Stomach", "Cells...EBV.transformed.lymphocytes", "Lung", "Adipose...Subcutaneous")

# Subset types of genes
selected_types <- c("lincRNA", "rRNA","snoRNA","antisense","miRNA","protein_coding","snRNA", "misc_RNA","pseudogene")

expression_data_small<-expression_data_3[expression_data_3$tissue %in% selected_tissues & expression_data_3$gene_type %in% selected_types, ]
expression_data_small$tissue <- factor(expression_data_small$tissue, labels = c("Adipose","Brain","Lymphocytes","Liver","Lung","Muscle","Stomach","Testis"))

write.table(expression_data_small, file = "expression_data.txt", sep = "\t", row.names = FALSE, quote = FALSE)

```



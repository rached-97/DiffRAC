# DiffRAC: A flexible R framework for comparing response proportions in high-throughput sequencing count data

A R framework to compare sequencing count ratios using a customized model matrix and DESeq2, from an experimental design, a formula and a read count tables. 

An example analysis can be found at `./examples`. 

# Inputs 

## design

The design data frame. Each row is one sample, and each column is an experimental variable. Sample names should be indicated as row.names, and the experimental variables as column names. Sample names have to match the column names in the count matrices (but not necessarily in the same order). The variables may be factors or numerical, and the user needs to make sure that the data has the intended class. There must be no NA values in the design. A minimum of two replicates per condition (or per combination of conditions) for the variable of interest should be used. For example:

| samples     | V1  | V2  |
| ----------- | --- | --- |
| cond1_rep1  |	1   | 0   |
| cond1_rep2  | 1   | 0   |
| cond2_rep1  |	1   | 1   |
| cond2_rep2  |	1   | 1   |
| cond3_rep1  |	0   | 0   |
| cond3_rep2  |	0   | 0   |
| cond4_rep1  |	0   | 1   |
| cond4_rep1  | 0   | 1   |

Another valid example:

| samples | cell_type      |                                
| ------- | -------------- |                              
| s1      |	cellLineA      |                          
| s2      |	cellLineA      |                            
| s3      |	cellLineB      |                         
| s4      |	cellLineB      |      

Which is equivalent to:

| samples | cellLineA |                                
| ------- | --------- |                              
| s1      |	0         |                          
| s2      |	0         |                            
| s3      |	1         |                         
| s4      |	1         |

Please avoid the use of "-" and "." in the sample names. Names should also not start with a number.

## formula

A `formula` indicating the relationship between the predictor and outcome variables. The variable names must be the same as in the design matrix. For example: 

\~ V1 + V2 + V1:V2


## count_num

The count matrix or count data frame for the numerator type. The rows and columns must match and be in the same order as the `count_denom` table. Column names should match the sample names in the design matrix (but not necessarily with the same order). Names should not start with a number. The row names must contain gene IDs.

| Gene_ID | cond1_rep1  | cond1_rep2  | cond2_rep1  | cond2_rep2  | cond3_rep1  | cond3_rep2  | cond4_rep1  | cond4_rep2  |
| ------- | --- | --- | --- | --- | --- | --- | --- | --- | 
| 22746   | 29  | 3   | 27  | 2   | 47  | 3   | 37  | 5   | 
| 235623  | 122 | 92  | 90  | 18  | 299 | 45  | 454 | 6   | 
| 238690  | 18  | 14  | 6   | 8   | 71  | 22  | 60  | 34   | 
| 330369  | 5   | 35  | 4   | 17  | 149 | 55  | 276 | 23   | 
| ...     | ... | ... | ... | ... | ... | ... | ... | ... | 

## count_denom

The count matrix or count data frame for the denominator type, similar to `count_num`.

## mode

Either `condition` or `sample`. Optionally, for large sample sizes, a condition-specific analysis can be performed, using the `condition` option, instead of a sample-specific investigation. This will significantly decrease the run time. The default is `mode="condition"`

## bias

The "bias" constant.  Optionally, the bias term can be supplied by the user. The default is `bias=1`

## optimizeBias

Optionally, an optimization can be performed to obtain the bias term, using `optimizeBias=T`. The default is `optimizeBias=F`.

# The DiffRAC function

## Usage

```r
DiffRAC_res <- DiffRAC(design, counts, formula, num, denom, mode)
```

## Output

Returns the customized model matrix and a DESeq dds object

DiffRAC returns a list with two elements:

The first element is the customized model matrix created by DiffRAC.

```r
DiffRAC_res$model_mat
```

The second element is a DESeq `dds` object (output from DESeqDataSetFromMatrix and DESeq functions). The user can then use their own contrasts and follow the DESeq2 documentation to get differential estimates (as a log2 fold-change) and identify differential events (such as differentially stabilized genes in this example).

```r
DiffRAC_res$dds
 ```
Please note that in case of factor variables, the variable names will be different from the inputed design and formula to the resulting dds object. For example, for a variable V1 with levels 0 and 1, a column V1 will be returned. For a variable V1 with levels "control" and "treatment", a column V1treatment will be returned. In the case of a variable V1 with levels "a", "b", and "c", then the design will contain V1b and V1c columns, each representing the change in stability relative to reference level "a". For numeric variables, the name will remain the same.

Differential events be identified by filtering for padj < 0.1, for example.



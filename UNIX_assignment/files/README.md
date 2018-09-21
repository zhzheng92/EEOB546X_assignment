# Workflow for data inspection and processing

## Data inspection
Check the data structure of both file: `fang_et_al_genotypes.txt` and `snp_position.txt`

```
cat fang_et_al_genotypes.txt
cat snp_position.txt

```
Since we only need three columns from `snp_position.txt`, i.e. "SNP_ID", "Chromosome" and "Position", only these three columns will be keeped in the later section of data processing.

## Data processing

1. Seperate maize and teosinte from `fang_et_al_genotypes.txt`

```
cat fang_et_al_genotypes.txt | grep -E "(ZMMIL|ZMMLR|ZMMMR)" > maize_genotypes.txt

cat fang_et_al_genotypes.txt | grep -E "(ZMPBA|ZMPIL|ZMPJA)" > teosinte_genotypes.txt
```

2. Transpose these two files 

####From now on, only use maize files as example, since operations for maize and teosinte files are the same

```
awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt


```

3. Remove header of the transposed files


```
sed "1,3d" transposed_maize_genotypes.txt > transposed_maize_genotypes_NH.txt

```

4. Select "SNP_ID", "Chromosome" and "Position" from `snp-position.txt`

```
awk '{print $1,$3,$4}' snp_position.txt > snp_position_cleaned.txt
```

5. Remove header


```
sed "1d" snp_position_cleaned.txt > snp_position_cleaned_NH.txt

```

6. Join tranposed genotype file and snp position file, change delimeter to tab

```
join -1 1 -2 1 snp_position_cleaned_NH.txt transposed_maize_genotypes_NH.txt > maize.all.tmp


```

7. Join the header of the genotype file and snp position file for later use 

```
head -n1 snp_position_cleaned.txt snp_position_cleaned.header

head -n1 transposed_maize_genotypes.txt > transposed_maize_genotype.header

sed 's/SAMPLE_ID/SNP_ID/g' transposed_maize_genotype.header

join -1 1 -2 1 snp_position_cleaned.header transposed_maize_genotype.header > maize_complete.header

```

8. Add header to `maize.all.tmp`

```
cat maize.all.tmp >> maize_complete.header
mv maize_complete.header maize.all.txt

```

9. Make file only contain specific chromome and sort according to requirements.
"PATTERN" is chromosome 1-10

```
# increasing position
awk -F'\t' '$2~/^PATTERN$/' maize.all.txt |  sort -n -k 3,3 > maize_PATTERN.tmp

# decreasing position and missing value as "-/-"
awk -F'\t' '$2~/^PATTERN$/' maize.all.txt |  sort -n -k 3,3 -r | sed 's/?/-/g'> maize_PATTERN_reverse_.tmp

```

10. Make file contain multiple and unknown position

```
awk -F'\t' '$2~/^multiple/|| $3~/^multiple/' maize.all.txt > maize_multiple.tmp

awk -F'\t' '$2~/^unknown/|| $3~/^unknown/' maize.all.txt > maize_unknown.tmp

```

11. Add headers to all the `.tmp` files in step 9 and step 10, as showed in step 8.

"SPECIES" below indicates maize or teosinte, in each folder in `EEOB546X_assignment/UNIX_assignment/files/SPECIES`

Now, all the `SPECIES_chrX.txt` are 10 files (1 for each chromosome) with SNPs ordered based on increasing position values and with missing data encoded by this symbol: ?;

All the `SPECIES_chrX_reversed.txt` are 10 files (1 for each chromosome) with SNPs ordered based on decreasing position values and with missing data encoded by this symbol: -;

All the `SPECIES_multiple.txt` and `SPECIES_unknown.txt`are 1 file with all SNPs with multiple positions in the genome and 1 file with all SNPs with unknown positions in the genome .


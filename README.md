# just_drugs
Drugs postagregator for longevity reporter in OakVar. It is a part of [Oakvar-Longevity](https://github.com/dna-seq/oakvar-longevity) module.


# Installation
## Installation through Oakvar

From Oakvar store:
```bash
ov module install just_drugs
```
Or make git clone of the current repository directly into "postaggregators" folder of Oakvar:
```bash
git clone https://github.com/dna-seq/just_drugs
```

## Manual installation

Create a folder with the name "just_drugs" in "postaggregators" folder of Oakvar and put it inside all the files from this git repository.

You can join our [discord](https://discord.gg/5WU6aSANXy) if you want to discuss the project or if you encountered any issues.


##  Detailed Description:

Our tool loads tables “var_drug_ann.tsv” (variant-drug associations) and “study_parameters.tsv” (quantitative effects data), merges them by sharing column “Variant Annotation ID” as an unambiguous index and produces a raw effect-variant-drug associations table. Currently, we preserve columns and "Variant Annotation ID", "Variant/Haplotypes", "Drug(s)", "Phenotype Category", "Significance" and "Sentence" for “var_drug_ann.tsv” table and columns "Variant Annotation ID", "Allele Of Frequency In Cases", "Allele Of Frequency In Controls", "P Value", "Ratio Stat Type", "Ratio Stat", "Confidence Interval Start" and "Confidence Interval Stop" for “study_parameters.tsv” table. Next we perform these steps of filtration:
1. remove entries with multiple “Variant/Haplotypes” values because we have not yet developed a way to unwind these cases clinically accurately, technically these entries have many “Variant/Haplotypes” values written in comma.
2. remove entries with multiple “Drug(s)” value because it is definitely just an imprecise formalization of information from a study and it is improbable that the effect for certain variants will be the absolutely same for different sets of drugs. These cases require more investigation and custom approaches for analysis.
3. remove entries with multiple “Phenotype Category” values, it is another case of imperfect study formalization.
4. exclude cases where “Drug” is actually a class of drug instead of a certain substance because the translation of the same effect value for the entire class (sometimes with unrelated substances) is dangerous in a clinical sense.
5. keep entries with RSID only variants and exclude entries with haplotypes – interpretation of haplotypes now is postponed due to the complexity of haplotypes.
6. remove cases where “stat type” is unknown because we can’t use a value of unknown statistical nature (possible stat types for PharmGKB are OR/RR/HR).
7. remove cases where the reference or alternative allele is unknown – we have no explanation why these cases are presented in PharmGKB.
8. remove cases where reference and the alternative allele is the same - we also have no explanation for why these cases are presented in PharmGKB.
After filtration we produce a clear effect-variant-drug associations table.

For analysis, we take input as table RSID:Allele/Allele, substitute these input alleles to the clear effect-variant-drug associations table, and produce substituted effect-variant-drug associations table where effect values the same if Allele from input matches with the reference allele and where effect values are inverted if Allele matches with the alternative allele (i.e. effect is 1.44 and if Allele is alternative, now effect will be 1/1.44 = 0.694). After the generation of the substituted table, we perform pivoting by each drug for integration of effects from different variants by simple multiplication (i.e. if we have 3 variants with effects 1.44, 0.54, and 1.22 for one drug – we will have 1.44 x 0.54 x 1.22 = 0,949 totally for this drug) it is based on assumption that different entities in the same category for the same drug measure the same biological phenomenon. Also, we separate these computations between different kinds of Phenotype Categories (i.e. all Odd Ratios and Relative Risks are multiplied separately also with separation between Metabolism/Pharmacokinetics and drug Efficacy). However, currently, the incompleteness of data and weakness of our algorithms don’t allow us to analyse heterozygosity.  By this simple approach we provide an integrated numerical estimation of genetic influence on the drug metabolism and efficacy.

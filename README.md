# Copy Number Variation 

This is a repo to compile and harmonizr various versions of CNV code that was originally used for mad4hatter. Should be made more flexible to allow for panel/ target alterations and incorporate various updates to method people have made. 

## Notes from meeting with Andres in Barcelona

**Currently**
* Current repo https://github.com/andres-ad/madh_utilities
    * Now forked and code is under `madh_utilities_estCNV/`
* Rmd needs to be updated 
* Loci info is for MH panel, theres a version for PfPHAST andres needs to push 
* Includes manifest example 
* PCA no one uses - was intended to be a clustering of runs into batches 
    * Not been properly benchmarked etc. 

**In andres pfphast rmd** 
* Need to set paths manually 
    * allele data 
    * manifest 
    * locus info
    * thresholds 
        * new ones for PfPHAST compared to MH  
* Locus info file (RDS) includes 
    * Amplicon info 
    * Locus filter - good for CNV. ones that amplify well  
    * Loci of interest 
        * Target (e.g. mdr1) and loci to use for it
* Manifest 
    * required : SampleID, Batch, SuperBatch, CNVControl (non duplicates)
* Code 
    * Loads everything 
    * Remove anything that’s not a Pf target 
    * Fit a Gamm, use residuals to normalise everything and find the DCR
* Outputs 
    * Html with all the code 
    * Some QC 
        * coverage
        * From the manifest - how many samples and controls are in the superbatch 
        * Plots Gamm for controls so you can see what it looks like 
        * For controls what is observed and black x is the median to normalise to 
            * Sanity check here that there isn’t something different happening between controls within a locus and there aren’t different profiles happening for batches across loci 
        * Use this to normalise and group loci (fitting GAMM) to estimate fold change of samples
        * HRP2 is fitted separately  because of different breakpoints 
            * 2 targets land on HRP, 2 targets are downstream of it 
            * Ones on hrp2 underperform - one is excluded from analysis
            * Both targets sitting on HRP2 and ones downstream need to be deleted to be classified as a deletion. 
                * Fit separately and take a maximum to decide 
        * Then final results are show with HRP2 collapsed as one target 
    * Fold change outputs for each gene 

**Yos**
* Took the one on github that Andres had adapted to take v0.1.8 outputs 
* Made R script callable on command line 
* Unsure what parasitemia is being used for 
* Can change thresholds and set outputs on command line 

**If people don’t have controls**
* Assume majority of samples have no CNV and use the median of the data for normalisation 
* This code is not in repo - Jessica and Andres have that 
* This should be added as a feature 
* This was found to work better (comments on slack from Bryan though in immrse channel)

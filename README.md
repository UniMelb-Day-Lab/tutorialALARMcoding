# _ALARM_coding 

## 1. SeekDeep Code: 

To Run SeekDeep in **Terminal**, load:

`export PATH=/home/user/software/SeekDeep-3.0.1/bin/:$PATH``

Run SeekDeep: 

`SeekDeep setupTarAmpAnalysis --samples sampleNamesPool1.tab.txt --outDir SeekDeep_Run --inputDir . --idFile multipleGenePairs.id.txt --overlapStatusFnp overlapStatuses.txt --refSeqsDir markerRefSeqs/forSeekDeep/refSeqs/ --extraExtractorCmds="--barcodeErrors 0 --midWithinStart 8 --primerWithinStart 8 --checkRevComplementForMids true --checkRevComplementForPrimers true" --extraProcessClusterCmds="--fracCutOff 0.035 --sampleMinTotalReadCutOff 100"`

## 2. MACSE Code: 

### Workflow to process sequences post-SeekDeep

1.  Unzip .gz files in `~/SeekDeep/Pool_X/Run_X/popClustering/marker/analysis/final/.` using this command:

`` for file in *.fastq.gz; do newfile=`echo "$file" | sed 's/.fastq.gz/.fastq/g'`; gunzip -c $file > $newfile; done ``

2.  Make file containing all fastq files:

`cat /your/path/to/folder/*.fastq > Pool_X_Marker.fastq`

3.  Convert fastq to fasta:

`paste - - - - < Pool_X_Marker.fastq | cut -f 1,2 | sed 's/^@/>/' | tr "\t" "\n" > Pool_X_Marker.fasta`

4.  Add Ref seq:

`cat ~/SeekDeep/MACSE/Marker_3D7_PlasmoDB.fasta Pool_X_Marker.fasta > Pool_X_Marker_All.fasta`

To add a space between the first file and the second use this:

`printf "\n" | cat ~/SeekDeep/MACSE/Marker_3D7_PlasmoDB.fasta - Pool_X_Marker.fasta > Pool_X_Marker_All.fasta`

### Running MACSE

To Run MACSE in **Terminal**, load:

-   `export PATH=/home/user/software/jre1.8.0_341/bin:$PATH`
-   `export PATH=/home/user/software/macse_v2.06.jar/bin:$PATH`

Run MACSE: `java -jar macse_v2.06.jar -prog alignSequences -seq test.fasta`

Then copy across the files to the computer local drive for input into R. 

## 3. Set of R Notebooks: 

Create the following folders: 

- **01_SeekDeep_Output**: Contains `extractionStats.tab.txt`, `selectedClustersInfo.tab.txt`, `extractionProfile.tab.txt` files for each pool and marker output from SeekDeep. This can be used at your own discretion to match processed sequences with read count data etc. 
- **02_MACSE_Output_AA**: Contains all `_AA.fasta` files output from MACSE.
- **02_MACSE_Output_NT**: Contains all `_NT.fasta` files output from MACSE.
- **03_DrugR_AA_Sequences**: Output of processed amino acid files created from `Step_01_MACSE_to_R.qmd`.
- **03_DrugR_NT_Sequences**: Output of processed nucleotide files created from `Step_01_MACSE_to_R.qmd`.
- **03_DrugR_AA_Changes**: Output of processed non-synonymous changes files created from `Step_01_MACSE_to_R.qmd`.
- **03_DrugR_NT_Changes**: Output of processed synonymous and non-synonymous changes files created from `Step_01_MACSE_to_R.qmd`.
- **04_Compiled_AA_Sequences**: Output of compiling all amino acid files together created from `Step_01_MACSE_to_R.qmd`.
- **04_Compiled_NT_Sequences**: Output of compiling all nucleotide files together created from `Step_01_MACSE_to_R.qmd`.
- **05_All_DrugR_AA_Sequences**: Output of cleaned amino acid files created from `Step_02_R_Cleaning.qmd`.
- **05_All_DrugR_NT_Sequences**: Output of cleaned nucleotide files created from `Step_02_R_Cleaning.qmd`.
- **05_All_DrugR_Haplotypes**: Output of cleaned condensed haplotype files created from `Step_02_R_Cleaning.qmd`.

## 4. Specific changes 

Both mdr1-2 and dhps-2 contain isolates that appear to have SAME genotype but are said to be in different clusters. I found that this is because for mdr1-2, either the last one or last six nucleotide differ. For dhps-2 there is a string of A's that are variable. 

- mdr1-2: Remove codons 1077, 1078, 1079

| Last 6 nucleotides      | n |
| ----------- | ----------- |
| AAAATG      | 1088       |
| AAAATT   | 981        |
| TTTCTG   | 550        |

- dhps-2: Remove codons 520 and 520.1

| Nucleotides 50 to 75      | n |
| ----------- | ----------- |
| TTAAAAAAAAAAAAAAACAAATTCT      | 2       |
| TTAAAAAAAAAAAAAACAAATTCTA   | 204        |
| TTAAAAAAAAAAAAACAAATTCTAT   | 18        |
| TTAAAAAAAAAAAACAAATTCTATA      | 1790       |
| TTAAAAAAAAAACAAATTCTATAGT   | 964        |
| TTAAAAAAAAAAACAAATTCTATAG   | 1        |

- NOTE: MID 47 (TGTGAGTAGT, [Roche](https://www.yumpu.com/en/document/read/24794008/roche-mid-adapters)) exists within the dhps-2 amplicon sequence. Our current SeekDeep parameters account for this by only searching for the MID in the first 8 nucleotides of the amplicon.  
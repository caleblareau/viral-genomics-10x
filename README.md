# Viral reactivation / latency using single-cell genomics

Pipelines for scRNA/scATAC-seq for viral latency/reactivation


## Viral reactivation (scRNA)
For each scRNA-seq library, pseudoalign R1 and R2 to `panref1.kb.idx` using the standard commands (these should be pretty stable; note that 8 threads are allocated in the `-t 8` step)

```
idx="viral-genomics-10x/viral-reactivation-scrna_v1/panref1.kb.idx"
fqpath="/path/to/fastq/"

for i in ALLO_Sample34 ALLO_Sample38 ALLO_Sample97 ALLO_Sample98
do
R1="${fqpath}/${i}_S1_L001_R1_001.fastq.gz"
R2="${fqpath}/${i}_S1_L001_R2_001.fastq.gz"
kallisto bus -i $idx -o "${i}_kb" -t 8 -x 10xv2 $R1 $R2
bustools sort -t 8 -o "${i}_kb/output_sorted.bus" "${i}_kb/output.bus" 
bustools text "${i}_kb/output_sorted.bus" -p > "${i}_pan.kb.txt"
done
```

Note the `-x` specifies the 10x chemistry version, which roughly is just parsing the barcode / UMI positions. 

Keep the `"${i}_pan.kb.txt"` file for downstream analysis. 


## Viral latency (scATAC)

Note that chromap, at least the version that I've been using, doesn't nicely handle the reverse complement R2 barcode, so have an appropriate whitelist handy...

```
cm="/path/to/chromap"
wl="/path/to/rev-737K-cratac-v1.txt"
idx="/path/to/viral-genomics-10x/viral-latency-scatac_v1/panref1.chromap.index"
fa="/path/to/viral-genomics-10x/viral-latency-scatac_v1/panref1.fasta"

fqpath="/path/to/fastq/"
ss=`ls $fqpath | grep R1 | sed 's/_R1_001.fastq.gz//g'`

for i in $ss
do 
r1="${fqpath}/${i}_R1_001.fastq.gz"
r2="${fqpath}/${i}_R2_001.fastq.gz"
r3="${fqpath}/${i}_R3_001.fastq.gz"

$cm --preset atac -x $idx -r $fa -1 $r1 -2 $r3 -o "${i}.bed" \
 -b $r2 --barcode-whitelist $wl -t 8 
done
```

This will produce a file `"${i}.bed"` that should be kept for downstream analysis. 
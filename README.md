# TBProfiler library for NC_018143.2

## How to generate the database files
These commands will show you how to generate the file present in this database from the [standard tbdb repo](https://github.com/jodyphelan/tbdb)
```
git clone git@github.com:jodyphelan/tbdb.git
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/277/735/GCF_000277735.2_ASM27773v2/GCF_000277735.2_ASM27773v2_genomic.fna.gz
gunzip -c GCF_000277735.2_ASM27773v2_genomic.fna.gz > genome.fasta
minimap2 genome.fasta ~/refgenome/MTB-h37rv_asm19595v2-eg18.fa --cs -cL > aln.paf

### Generating Barcode
paftools.js liftover aln.paf tbdb/barcode.bed | sed 's/Chromosome/NC_018143.2/' > tmp.bed
cut -f1-3 tmp.bed> f1
cut -f4-10 tbdb/barcode.bed > f2
paste f1 f2 > barcode.bed

### Generating GFF
cat ~/github/tbdb/genome.gff | grep -v '#'  | cut -f1-3 > f1
cat ~/github/tbdb/genome.gff | grep -v '#'  | cut -f6-9 > f3
cut -f4,5 ~/github/tbdb/genome.gff | grep -v '#'  | awk '{print "Chromosome\t"$1"\t"$2}' > tmp.bed
paftools.js liftover aln.paf tmp.bed | cut -f2,3 > f2
paste f1 f2 f3 | sed 's/Chromosome/NC_018143.2/' > genome.gff

### Generating genes file
cut -f1,2 ~/github/tbdb/genes.txt > f1
cut -f5,6 tbdb/genes.txt > f3
cut -f3-4 ~/github/tbdb/genes.txt  | awk '{print "Chromosome\t"$1"\t"$2}' > tmp.bed
paftools.js liftover aln.paf tmp.bed | cut -f2,3 > f2
paste f1 f2 f3 > genes.txt

### Generate library
cp tbdb/tbdb.csv .
sed 's/Chromosome/NC_018143.2/' tbdb/parse_db.py > parse_db.py
python parse_db.py -p NC_018143.2

### Load library and test
tb-profiler load_library NC_018143.2
tb-profiler profile -1 /path/to/reads --db NC_018143.2 -t 4  --txt
```

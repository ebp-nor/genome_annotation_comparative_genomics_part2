### 00 - Let's begin by setting up our environment:
```
conda activate selection
```

### 01 - first let's move to the right folder and make a proper folder structure
```
### Copy and paste the following: We make an environment at our work space (first command), move there (second command) and create a folder structure (third command)
mkdir -p /cluster/work/projects/ec146/dnds/$USER
cd /cluster/work/projects/ec146/dnds/$USER
mkdir 00_data  01_alignments  02_zorro  03_tree  04_absrel
```

### 02 - Now, let's fetch the data we need.
```
cp /projects/ec146/work/jose/data/*fa ./00_data

# You can take a look at it:
less ./00_data/*fa
```

### 03 - Let's make an alignment with Prank
```
# This command uses prank to align, moves the file to the right location, and removes the tmp folder
prank -d=00_data/OG0000246.fa -o=OG0000246 -codon -verbose; mv OG0000246.best.fas 01_alignments/OG0000246.fai; rm -r tmp*

# Notice we have another OG. "OG0000251.fa". Run prank with the same command but with the 2nd OG
prank -d=00_data/OG0000251.fa -o=OG0000251 -codon -verbose; mv OG0000251.best.fas 01_alignments/OG0000251.fai; rm -r tmp*
```

### 04 - We need to certify the alignments are high-quality. We use Zorro for this.
```
# This command runs zorro on both alingment and estimates a alignment quality threshold. From the abstract: "Here we describe ZORRO, a probabilistic masking program that accounts for alignment uncertainty by assigning confidence scores to each alignment position.".
for i in 01_alignments/*fai; do zorro $i > 02_zorro/${i#01_alignments/}.zorro; done

# To see an average score (I usually go with > 5)
for i in 02_zorro/*zorro; do echo $i; awk '{ sum += $1 } END { if (NR > 0) print sum / NR }' $i; done
```

### 05 - We need a tree for hyphy.
```
# The second command just moves things.
for i in 01_alignments/*fai; do iqtree2 -s $i -B 1000 -T 1; mv ${i}.* ./03_tree; done
```

### 06 - One of the sequences has a remove codon (OG0000251.fai)
```
cd 01_alignments
hyphy rmv
# Now you'll get a screen. You'll have to click "1" for Universal code, copy and paste the file name "OG0000251.fai", and specify we only want to remove stop codons "1", and save the file "OG0000251.fai"
cd ..
```
## 07 - We are finally ready for hyphy
```
for i in OG0000246.fai  OG0000251.fai; do hyphy absrel --alignment 01_alignments/$i --tree 03_tree/${i}.treefile > 04_absrel/${i%.fai}.hyphy; echo "$i done"; mv 01_alignments/${i}.ABSREL.json 04_absrel/${i}.ABSREL.json; done

# Explore the output and we will discuss it later.
```

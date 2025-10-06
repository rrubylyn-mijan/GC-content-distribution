# GC-content-distribution
This workflow calculates GC content distribution across the Sumai 3, Rollag, and Glenn wheat genomes using BEDTools.
The resulting data can be used for Circos visualization or comparative GC composition analysis between subgenomes.

## 1. Make a directory
```bash
mkdir gc-content-distribution

cd gc-content-distribution
```

## 2. Generate 10 kb Windows and Compute GC Content
```bash
#!/bin/bash
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -p atlas
#SBATCH --mem=100GB
#SBATCH -J gc_content_glenn
#SBATCH -A genolabswheatphg

ml bedtools2/2.31.1

bedtools makewindows -g /directory/this/saved/wheat.fasta.fai -w 10000 > wheat-windows.bed
bedtools nuc -fi /directory/this/saved/wheat.fasta -bed wheat-windows.bed > gc-content-wheat.bed
```

## 3. Extract GC Density Columns
```bash
cut -f 1,2,3,5 gc-content-wheat.bed > gc-content-wheat-density-extracted.bed
```

## 4. Calculate GC Content Range
```bash
awk 'NR > 1 {
    if (NR == 2) { min = $4; max = $4 }
    if ($4 > max) max = $4
    if ($4 < min) min = $4
}
END { print "Max GC%:", max, "\nMin GC%:", min }' gc-content-wheat-density-extracted.bed
# Example Output:
# Max GC%: 0.699800
# Min GC%: 0.184000
```

## 5. Frequency Distribution of GC Content
```bash
awk 'NR > 1 {
    bin_size = 5
    bin = int($4 / bin_size) * bin_size
    freq[bin]++
}
END {
    for (b in freq) {
        print b "-" b + bin_size "%:", freq[b]
    }
}' x_sumai3_gc_content_density_extracted.bed | sort -n

awk '{print $4}' gc-content-wheat-density-extracted.bed | sort -n | uniq -c
```

## 6. Identify Low and High GC Regions
```bash
awk 'NR > 1 {
    if ($4 < 0.4) low++
    else if ($4 > 0.6) high++
}
END {
    print "Low GC content count (< 40%):", low
    print "High GC content count (> 60%):", high
}' gc-content-wheat-density-extracted.bed
```

## 7. Rename Chromosomes for Circos Compatibility
```bash
sed 's/^chr/ta/' gc-content-wheat-density-extracted.bed > x8-gc-content-wheat-density-extracted.bed
```

## 8. Calculate GC Content per Chromosome
```bash
awk '
{
    chr = $1
    gc = $4
    sum_gc[chr] += gc
    count[chr] += 1
}
END {
    print "Chromosome\tTotal_GC_Content\tAverage_GC_Content"
    for (chr in sum_gc) {
        avg_gc = sum_gc[chr] / count[chr]
        print chr "\t" sum_gc[chr] "\t" avg_gc
    }
}' gc-content-wheat-density-extracted.bed
```


Maintainer:

Ruby Mijan

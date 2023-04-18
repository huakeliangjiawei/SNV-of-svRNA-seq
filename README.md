# SNV-of-svRNA-seq

soft ware
1. cellsnp:https://github.com/single-cell-genetics/cellSNP
2. sinto  :https://github.com/timoast/sinto

split -l 1000 barcoeds.tsv -d -a 3 url_

#!bin/bash
##此时的barcodes文件应该有两列
cat bars.txt|while read id
do
	dirs=${id%%.*}
	mv $id $dirs
done

#!/bin/bash

#SBATCH -p intel-sc3           # intel-sc3 partition
#SBATCH -q normal             # normal qos
#SBATCH -J sinto_L22              # job name
#SBATCH -c 24                 # the numbers of cores

cat bars.txt|while read id
do
        cd $id/
        bars_code=${id}.tsv
        sinto filterbarcodes -b /storage/shihongjunLab/lichenxi/liangjiawei/L118016/splitbam/splitbar/vcf_outs/L118016_Aligned.sortedByCoord.out.bam.featureCounts.bam -c $bars_code --barcodetag "CB" -p 24
        cd ../
done



#!/bin/bash

#SBATCH -p intel-sc3           # intel-sc3 partition
#SBATCH -q normal             # normal qos
#SBATCH -J samindex              # job name
#SBATCH -c 26                 # the numbers of cores

cat bars.txt|while read id
do
        cd $id/
        ls *.bam>1
        cat 1|while read ids
        do
                dir=${ids%%.*}
                mkdir $dir
                mv $ids $dir
                cd $dir
                samtools index $ids
                cd ../
        done
        for file in `ls -d */`
        do
                echo $file >>tmp
        done
        cut tmp -d"/" -f1>>dir.txt
        cd ../
done


#!bin/bash

cat bars.txt|while read id
do
        cd $id
        rm *.sh
        bar=${id}.tsv
        ids=ids
        BARCODE=/storage/shihongjunLab/lichenxi/liangjiawei/L118016/splitbam/splitbar/$id/$bar
        echo "#!/bin/bash

#SBATCH -p amd-ep2-short
#SBATCH -q huge
#SBATCH -J samindex
#SBATCH -c 4

cat dir.txt|while read ids
do
        cd \$ids/
        BAM=\${ids}.bam
        OUT_DIR=\${ids}_out
        mkdir \${OUT_DIR}
        cellsnp-lite -s \$BAM -b $BARCODE -O \$OUT_DIR -p 4 --minMAF 0.1 --minCOUNT 100 --chrom 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19
        cd ../
done">>${id}.sh
        cd ../
done


#!bin/bash

cat bars.txt|while read id
do
        cd $id/
        sbatch ${id}.sh
        cd ../
done

#!bin/bash

cat bars.txt|while read id
do
        cd $id/
        cat dir.txt|while read ids
        do
                cd $ids/${ids}_out/
                new_id=${ids}_1.vcf
                mv cellSNP.base.vcf $new_id
                cp $new_id ../../../allvcfs
                cd ../../
        done
        cd ../
done

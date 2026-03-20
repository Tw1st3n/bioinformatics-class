# homework 1-3
## 作业说明：
参考和学习本章内容，写出一个 bash 脚本，可以使它自动读取一个文件夹（例如 bash_homework/）的内容，将该文件夹下文件的名字输出到 filenames.txt, 子文件夹的名字输出到 dirname.txt 。<br>
## 提示和要求：
参考和学习本章内容，写出一个 bash 脚本，可以使它自动读取一个文件夹（例如 bash_homework/）的内容，将该文件夹下文件的名字输出到 filenames.txt, 子文件夹的名字输出到 dirname.txt 。<br>
> bash 脚本
```vim
#!/bin/bash
CUR_DIR=`ls`
echo $CUR_DIR
files_output="filenames.txt"
dirs_output="dirname.txt"
> "$files_output"
> "$dirs_output"
for val in $CUR_DIR
do
        if [ -f "$val" ];then
                echo "$val" >> $files_output
        elif [ -d "$val" ];then
                echo "$val" >> $dirs_output
        fi
done
echo "filenames.txt:"
cat "$files_output"
echo "dirname.txt:"
cat "$dirs_output"
exit 0
```
> filenames.txt
```vim
a1.txt
a.txt
b1.txt
bam_wig.sh
b.filter_random.pl
c1.txt
chrom.size
c.txt
d1.txt
dirname.txt
dir.txt
e1.txt
f1.txt
filenames.txt
human_geneExp.txt
if.sh
image
insitiue.txt
mouse_geneExp.txt
name.sh
name.txt
number.sh
out.bw
random.sh
read.sh
test3.sh
test4.sh
test.sh
test.txt
typescript
wigToBigWig
```
>dirname.txt
```vim
a-docker
app
backup
bin
biosoft
c1-RBPanno
datatable
db
download
e-annotation
exRNA
genome
git
highcharts
home
hub29
ibme
l-lwl
map2
mljs
module
mogproject
node_modules
perl5
postar2
postar_app
postar.docker
RBP_map
rout
script
script_backup
software
tcga
test
tmp
tmp_script
var
x-rbp
```

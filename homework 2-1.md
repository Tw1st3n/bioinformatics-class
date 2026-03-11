# 以test_command.gtf文件为例练习linux操作
启动docker后，打开powershell <br>

**docker  exec -it -u root liuhao_linux bash**进入linux操作环境<br>

输入**cd share** 打开test_command.gtf文件所处的文件夹
## 统计文件的行数以及字符数
**输入命令**：wc test_command.gtf<br>
**输出结果**：8  96 636 test_command.gtf<br>
## 利用 grep 等命令尝试筛选并输出示例文件中以 chr_ 起始，并且基因id为 YDL248W 的行
**输入命令**：grep '^chr_' test_command.gtf | grep 'YDL248W'<br>
**输出结果**：<br>
chr_IV  ensembl gene    1802    2953    .       +       .       gene_id "YDL248W"; gene_version "1";<br>
chr_IV  ensembl transcript      802     2953    .       +       .       gene_id "YDL248W"; gene_version "1";<br>
chr_IV  ensembl start_codon     1802    1804    .       +       0       gene_id "YDL248W"; gene_version "1";<br>
## 利用 sed 等命令将示例文件中的 chr_ 替换为 chromosome_ 并输出每行的第1，3，4，5列。（无需改动原文件，只输出结果）
**输入命令**：sed 's/chr_/chromosome_/g' test_command.gtf | awk '{print $1, $3, $4, $5}'<br>
**输出结果**：<br>
chromosome_IV gene 1802 2953<br>
chromosome_IV transcript 802 2953<br>
chromosome_IV exon 1802 2953<br>
chromosome_IV CDS 1802 950<br>
chromosome_IV start_codon 1802 1804<br>
chromosome_IV stop_codon 2951 2953<br>
chromosome_IV gene 762 3836<br>
chromosome_IV transcript 3762 836<br>
## 通过man命令以及更多的资料学习简单的 awk 命令，尝试互换示例文件的第2列和第3列，并且对输出结果利用 sort 命令依照第4和第5列数字大小排序，将最终结果输出到result.gtf文件中
**输入命令**：awk '{ temp = $2; $2 = $3; $3 = temp; print }' test_command.gtf | sort -k4,4n -k5,5n > result.gtf<br>
**输出文件result.gtf**：cat result.gtf<br>
chromosome_IV gene ensembl 762 3836 . + . gene_id "YDL247W-A"; gene_version "1";<br>
chr_IV transcript ensembl 802 2953 . + . gene_id "YDL248W"; gene_version "1";<br>
chromosome_IV CDS ensembl 1802 950 . + 0 gene_id "YDL248W"; gene_version "1";<br>
chr_IV start_codon ensembl 1802 1804 . + 0 gene_id "YDL248W"; gene_version "1";<br>
chr_IV gene ensembl 1802 2953 . + . gene_id "YDL248W"; gene_version "1";<br>
chromosome_IV exon ensembl 1802 2953 . + . gene_id "YDL248W"; gene_version "1";<br>
chromosome_IV stop_codon ensembl 2951 2953 . + 0 gene_id "YDL248W"; gene_version "1";<br>
chr_IV transcript ensembl 3762 836 . + . gene_id "YDL247W-A"; gene_version "1";<br>
## 更改示例文件的权限，使得文件所有者及所在用户组用户可读、写、执行而其他用户只可读，展示权限修改前后的权限变化。
**输入命令**：ls -l test_command.gtf<br>
**输出结果**：-rwxrwxrwx 1 root root 636 Mar 11 03:28 test_command.gtf<br>
**输入命令**:chmod 774 test_command.gtf（注意需要root权限进入）<br>
**输入命令**：ls -l test_command.gtf<br>
**输出结果**：-rwxrwxr-- 1 root root 636 Mar 11 03:28 test_command.gtf<br>

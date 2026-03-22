# 序列分析BLAST

---

## 1. 网页版BLASTP比对操作与E值/P值解释

### 操作截图
![]()


### E值与P值的实际意义
- **E值（Expect value）**：  
  表示在随机背景下，数据库中出现与当前匹配得分相同或更高的序列的**期望次数**。E值越小，匹配显著性越高（如E=0.01表示随机匹配的期望次数为0.01，结果更可靠）。  
- **P值（Probability value）**：  
  表示当前匹配结果由随机因素导致的**概率**。P值越小，结果越显著（但BLAST中更常用E值，因E值结合了数据库大小的影响，更具可比性）。


---

## 2. Bash脚本实现序列打乱与BLAST比对

### 1. Bash脚本（`shuffle_blast.sh`）
```bash
#!/bin/bash

# -------------------------- 1. 定义原序列与输出文件 --------------------------
original_seq="MSTRSVSSSSYRRMFGGPGTASRPSSSRSYVTTSTRTYSLGSALRPSTSRSLYASSPGGVYATRSSAVRL"
fasta_file="shuffled_proteins.fasta"  # 存储10条打乱序列
result_file="blast_pairs_results.txt"  # 存储BLAST比对结果

# -------------------------- 2. 生成10条随机打乱的蛋白序列 --------------------------
echo "正在生成10条随机打乱的蛋白序列..."
python3 << END > "$fasta_file"
import random

# 原序列
seq = "$original_seq"
# 生成10条打乱序列（FASTA格式）
for i in range(10):
    seq_list = list(seq)
    random.shuffle(seq_list)  # 随机打乱序列
    shuffled_seq = "".join(seq_list)
    print(f">shuffled_{i+1}")  # FASTA序列ID
    print(shuffled_seq)
END

# -------------------------- 3. 构建BLAST数据库（蛋白序列） --------------------------
echo "正在构建BLAST数据库..."
makeblastdb -in "$fasta_file" -dbtype prot -out shuffled_db -logfile makeblastdb.log

# -------------------------- 4. 两两BLAST比对（输出表格格式） --------------------------
echo "正在进行两两BLAST比对..."
blastp -query "$fasta_file" \
       -db shuffled_db \
       -out "$result_file" \
       -outfmt "6 qseqid sseqid pident length evalue bitscore" \
       -evalue 10  # 宽松E值阈值以展示随机匹配结果

echo "完成！结果文件：$result_file"
```


### 2. 脚本使用说明
- 依赖：安装 `ncbi-blast+`（Ubuntu: `sudo apt install ncbi-blast+`；CentOS: `sudo yum install ncbi-blast+`）。  
- 运行：  
  ```bash
  chmod +x shuffle_blast.sh  # 赋予执行权限
  ./shuffle_blast.sh          # 运行脚本
  ```


### 3. 结果示例与解释
**结果文件（`blast_pairs_results.txt`）示例内容**：
```
shuffled_1  shuffled_1  100.00  50  0.0   100.0
shuffled_1  shuffled_2  20.00   50  5.2   15.3
shuffled_2  shuffled_3  18.00   50  8.1   12.5
...
```

**结果解释**：
- 第一行：序列与自身比对，一致性100%，E值接近0（完全匹配）。  
- 其他行：随机打乱的序列两两比对，一致性低（~20%）、E值高（>1），说明匹配是随机产生的，无生物学同源性。


---

## 3. BLAST提高速度的核心方法（除动态规划外）

BLAST使用 **“种子-扩展法（Seed-and-Extend）”** 大幅提升速度，原理如下：

### 方法步骤
1. **种子查找**  
   将查询序列拆分为短“种子”（蛋白序列通常为3个氨基酸，核酸为11个碱基），在数据库中快速定位与种子**完全匹配**的位置。  
2. **无间隙扩展**  
   从种子位置向两侧延伸，不允许插入缺失（gap），直到匹配得分低于阈值，快速排除无意义匹配。  
3. **有间隙精细比对**  
   对高分候选区域，使用动态规划（如Smith-Waterman）进行有间隙的精细比对，得到最终结果。


### 为什么能提高速度？
全序列动态规划的时间复杂度为 **O(m×n)**（m、n为两条序列长度），而种子-扩展法通过：  
- 先过滤掉99%以上无种子匹配的无关序列；  
- 仅对少数候选区域做耗时的动态规划；  
将实际计算量降低几个数量级，因此速度远快于纯动态规划。


---

## 4. 对称与不对称PAM250矩阵的差异

### 差异原因（基于“Symmetry of the PAM matrices”文献逻辑）
PAM矩阵基于“可接受点突变”概率构建，对称与不对称的核心区别在于**进化可逆性假设**：

- **对称PAM250**：  
  假设进化过程是**可逆的**（即氨基酸A→B的突变概率 = B→A的概率）。构建时通过数学对称化处理（如取双向突变率的平均值），使矩阵满足 `PAM[i][j] = PAM[j][i]`。  

- **不对称PAM250**：  
  基于**真实进化数据**（如同源蛋白家族的突变统计），不假设可逆性。实际中，不同氨基酸的突变率可能不对称（例如：丝氨酸有6个密码子，突变为丙氨酸的概率可能高于丙氨酸→丝氨酸），因此矩阵 `PAM[i][j] ≠ PAM[j][i]`。


### 应用上的不同
- **对称PAM250**：  
  适用于**常规序列比对**（如BLAST默认场景）。对称矩阵计算简单、速度快，且在大多数进化距离下能满足同源性检测需求。  

- **不对称PAM250**：  
  适用于**精细进化分析**（如特定物种类群的进化树构建、密码子偏好性强的序列比对）。它能更真实地反映突变方向的差异，但需更多进化数据支撑，计算复杂度更高，应用场景较窄。


如需进一步验证PAM矩阵文献细节或脚本调试，可随时补充说明！

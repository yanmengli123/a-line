# A-LINER 中文使用教程

本文档介绍如何在 Windows 环境中安装和使用 `a-liner`，包括输入文件准备、常用命令、官方示例、参数说明、Python 调用方式和常见问题。

## 1. A-LINER 是什么

`a-liner` 是一个用于比较基因组可视化的 Python 命令行工具。它将不同序列区域绘制为水平轨道，并可在轨道之间显示序列比对关系。

它支持：

- 序列区间和正负链方向；
- 自定义线性比对文件；
- BLASTN `outfmt 6`；
- minimap2 PAF；
- LASTZ；
- MUMmer `show-coords`；
- GFF3、GenBank 和 Excel 格式的基因注释；
- 基因箭头和基因名称；
- 区域高亮；
- GC 含量、拷贝数等逐位置散点数据；
- 按序列一致性着色；
- 输出矢量 PDF。

程序的主要处理流程是：

```text
读取序列配置
    ↓
计算轨道位置和画布尺寸
    ↓
绘制序列、比例尺和名称
    ↓
绘制序列比对
    ↓
绘制基因、区域高亮和散点数据
    ↓
输出 PDF
```

## 2. 当前电脑上的环境

本项目位于：

```text
C:\Users\32110\Desktop\a-liner-main
```

推荐使用的 Python 解释器：

```text
D:\soft\Python310\python.exe
```

命令行程序位于：

```text
D:\soft\Python310\Scripts\a-liner.exe
```

当前项目已经以 editable 模式安装到这个 Python 3.10 环境中。也就是说，修改项目文件夹里的 Python 源码后，`a-liner.exe` 会直接使用修改后的源码。

在 PowerShell 中检查程序：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' --version
& 'D:\soft\Python310\Scripts\a-liner.exe' --help
```

如果第一条命令输出类似下面的内容，说明命令行入口可以使用：

```text
a-liner v.1.1.0
```

## 3. 项目目录结构

```text
a-liner-main/
├── a_liner/                   # Python 核心源码
│   ├── cli.py                 # 命令行入口和绘图总流程
│   ├── common.py              # 参数、尺寸计算和公共函数
│   ├── seqs.py                # 序列配置、轨道和比例尺
│   ├── alignment.py           # 各种比对格式
│   ├── genes.py               # GFF3、Excel、GenBank 注释
│   ├── highlight.py           # 区域高亮
│   └── scatterplot.py         # 散点轨道
├── sample_data/
│   ├── a_Stx-phages/          # 细菌噬菌体局部比较示例
│   └── b_ostrich-emu/         # 鸵鸟和鸸鹋染色体比较示例
├── docs/images/               # README 使用的示例图片
├── pyproject.toml             # 包信息和依赖
└── README.md                  # 英文说明
```

## 4. 重新安装或更新安装

如果以后更换 Python 环境，可以在 PowerShell 中执行：

```powershell
Set-Location 'C:\Users\32110\Desktop\a-liner-main'

& 'D:\soft\Python310\python.exe' -m pip install -e .
```

`-e` 表示 editable 安装。

如果缺少依赖，可以执行：

```powershell
& 'D:\soft\Python310\python.exe' -m pip install `
  matplotlib `
  pandas `
  biopython `
  numpy `
  openpyxl `
  bcbio-gff
```

然后再次检查：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' --help
```

## 5. 最简单的使用方式

`a-liner` 唯一必需的输入是序列配置文件。即使没有序列比对或基因注释，也可以只绘制序列轨道。

### 5.1 创建序列配置

创建一个名为 `sequence_config.txt` 的制表符分隔文件：

```text
n	ID	start	end	strand	name
0	sequence_A	1	100000	+	物种 A
1	sequence_B	1	90000	+	物种 B
```

注意：

- 文件必须包含表头；
- 分隔符必须是 Tab，不能只是若干空格；
- `n` 从 `0` 开始；
- `n` 必须连续，例如 `0、1、2、3`；
- `0` 是图中最下面的轨道；
- `ID` 必须与比对、GFF3、高亮和散点文件中的序列 ID 一致；
- `start` 和 `end` 使用 1-based 闭区间坐标；
- `strand` 只能是 `+` 或 `-`；
- `name` 是图中显示的名称。

### 5.2 只绘制序列轨道

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --out 'my_sequences'
```

程序会在当前目录生成：

```text
my_sequences.pdf
```

如果省略 `--out`，默认输出：

```text
out.pdf
```

## 6. 序列配置文件

### 6.1 TSV 格式

通过 `-i` 或 `--input` 指定：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --out 'result'
```

标准结构：

```text
n	ID	start	end	strand	name
0	chr1	1	200000	+	样本 A chr1
1	chr1	50000	250000	-	样本 B chr1
```

同一轨道也可以包含多段序列。相同 `n` 的记录会按照文件中的顺序从左到右排列：

```text
n	ID	start	end	strand	name
0	contig_1	1	50000	+	Contig 1
0	contig_2	1	30000	+	Contig 2
1	chromosome	1	100000	+	Reference
```

### 6.2 Excel 格式

Excel 文件也必须包含以下列：

```text
n | ID | start | end | strand | name
```

使用方法：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  --xlsx 'sequence_config.xlsx' `
  --xlsx_sheet 'Sheet1' `
  --out 'result'
```

如果使用第一个工作表，可以不写 `--xlsx_sheet`。

### 6.3 反向显示序列

将 `strand` 设置为 `-`：

```text
1	sequence_B	1	90000	-	反向显示的物种 B
```

程序会反转该序列的横坐标。该序列上的基因箭头方向也会相应转换。

### 6.4 留出空白区域

如果 `name` 为 `BLANK`，该记录会作为布局空白存在，但不会绘制序列线和名称：

```text
0	gap	1	20000	+	BLANK
```

## 7. 添加序列比对

### 7.1 自定义比对格式

使用 `-a` 或 `--alignment`。

文件没有表头，每行七列：

```text
seq_ID1	start1	end1	seq_ID2	start2	end2	identity
```

示例：

```text
sequence_A	1000	12000	sequence_B	2000	13000	98.5
sequence_A	20000	35000	sequence_B	40000	25000	92.1
```

第二条记录的 `start2 > end2` 表示反向比对。

运行：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --alignment 'alignment.txt' `
  --min_identity 80 `
  --alignment_alpha 0.5 `
  --out 'alignment_result'
```

### 7.2 BLASTN

先生成 BLASTN `outfmt 6`：

```bash
makeblastdb -in reference.fa -dbtype nucl
blastn -query query.fa -db reference.fa -outfmt 6 -out blastn.txt
```

然后绘图：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --blastn 'blastn.txt' `
  --min_identity 80 `
  --min_alignment_len 1000 `
  --out 'blastn_result'
```

BLASTN 文件中的 `qseqid` 和 `sseqid` 必须能在序列配置文件的 `ID` 列中找到。

### 7.3 minimap2

建议使用 `-c`，使 PAF 文件包含 CIGAR 字段：

```bash
minimap2 -c reference.fa query.fa > alignment.paf
```

绘图：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --minimap2 'alignment.paf' `
  --min_identity 85 `
  --min_alignment_len 10000 `
  --out 'minimap2_result'
```

当前代码使用 PAF 中的 `cg:Z:` CIGAR 字段估算一致性。如果 PAF 没有该字段，比对可能因为一致性被计算为 0 而不显示。

### 7.4 LASTZ

生成文件：

```bash
lastz reference.fa query.fa \
  --format=general \
  --output=lastz.txt
```

绘图：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --lastz 'lastz.txt' `
  --out 'lastz_result'
```

### 7.5 MUMmer

生成坐标文件：

```bash
nucmer --prefix output_nucmer reference.fa query.fa
show-coords -H output_nucmer.delta > show-coords.tsv
```

绘图：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --mummer 'show-coords.tsv' `
  --out 'mummer_result'
```

### 7.6 同时加载多个比对文件

一个参数后面可以跟多个文件：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --blastn 'part1.txt' 'part2.txt' 'part3.txt' `
  --out 'combined_result'
```

默认只绘制相邻轨道之间的比对。需要跨越非相邻轨道时添加：

```text
--include_nonadjacent
```

## 8. 添加基因注释

### 8.1 GFF3

默认绘制 `gene` 类型：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --gff3 'annotation.gff3' `
  --out 'gff_result'
```

绘制 CDS 和 pseudogene：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --gff3 'annotation.gff3' `
  --feature CDS pseudogene `
  --gene_label_attr gene `
  --out 'gff_result'
```

GFF3 的第一列 `seqid` 必须匹配序列配置中的 `ID`。

支持标准九列 GFF3：

```text
seqid	source	type	start	end	score	strand	phase	attributes
```

也支持第十列直接指定 Matplotlib 颜色：

```text
seqid	source	type	start	end	score	strand	phase	attributes	#FF0000
```

### 8.2 GenBank

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --gb 'annotation.gb' `
  --feature CDS `
  --gene_label_attr gene `
  --out 'genbank_result'
```

GenBank 记录的 `record.id` 必须匹配序列配置中的 `ID`。

### 8.3 Excel 注释

如果注释数据是没有表头的 GFF 风格 Excel：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --gff_xlsx 'annotation.xlsx' `
  --feature CDS `
  --out 'excel_annotation_result'
```

### 8.4 根据基因名称自动着色

准备一个制表符分隔文件 `feature_colors.txt`，没有表头：

```text
Shiga toxin	stxA/stxB	#E41A1C
phage genes	capsid/tail/portal	#377EB8
integrase	integrase/int	#FFAA00
```

三列分别是：

1. 图例名称；
2. 正则表达式或关键词，多个表达式用 `/` 分隔；
3. Matplotlib 颜色。

使用：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --gff3 'annotation.gff3' `
  --feature CDS `
  --feature_color_map 'feature_colors.txt' `
  --gene_label_attr gene `
  --out 'colored_genes'
```

## 9. 添加区域高亮

高亮文件没有表头，每行四列：

```text
seq_ID	start	end	color
```

示例：

```text
sequence_A	10000	25000	#FF9999
sequence_B	30000	45000	lightblue
```

运行：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --highlight 'highlights.txt' `
  --h_alpha 0.4 `
  --h_thickness 4 `
  --out 'highlight_result'
```

可以使用 Matplotlib 颜色名称或十六进制颜色，例如：

```text
red
lightblue
#FF0000
#66CCFF
```

## 10. 添加散点轨道

散点文件没有表头，每行三列：

```text
seq_ID	position	value
```

例如：

```text
sequence_A	1000	45.3
sequence_A	2000	47.1
sequence_A	3000	52.8
sequence_B	1000	41.2
```

绘制 GC 含量：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --scatter 'gc_content.txt' `
  --scatter_min 30 `
  --scatter_max 70 `
  --scatter_ylines 30 50 70 `
  --marker_color '#C33399' `
  --marker_size 3 `
  --background_color white `
  --out 'gc_result'
```

参数说明：

- `--scatter_min`：纵轴最小值；
- `--scatter_max`：纵轴最大值；
- `--scatter_ylines`：水平参考线；
- `--scatter_space`：散点轨道相对高度；
- `--marker_color`：散点颜色；
- `--marker_size`：散点大小；
- `--marker_style`：散点形状；
- `--background_color`：散点轨道背景。

高亮散点背景：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --scatter 'copy_number.txt' `
  --sp_highlight 'highlights.txt' `
  --sp_h_alpha 0.3 `
  --out 'scatter_highlight_result'
```

## 11. 一个完整的组合示例

下面的命令同时显示 BLASTN、GFF3、基因颜色、高亮和 GC 含量：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --seq_color black `
  --seq_thickness 0.8 `
  --scale tick `
  --blastn 'blastn.txt' `
  --min_identity 80 `
  --min_alignment_len 500 `
  --alignment_alpha 0.35 `
  --colormap 0 `
  --gff3 'annotation.gff3' `
  --feature CDS pseudogene `
  --feature_color_map 'feature_colors.txt' `
  --gene_label_attr gene `
  --gene_thickness 6 `
  --gene_edge_color black `
  --highlight 'highlights.txt' `
  --h_alpha 0.5 `
  --h_thickness 8 `
  --scatter 'gc_content.txt' `
  --scatter_min 30 `
  --scatter_max 70 `
  --scatter_ylines 30 50 70 `
  --marker_color '#C33399' `
  --background_color white `
  --out 'complete_result'
```

## 12. 运行官方示例

官方示例使用 `.sh` 脚本。Windows 下建议使用 Git Bash。

### 12.1 示例一：细菌 Stx 噬菌体

在 Git Bash 中：

```bash
export PATH="/d/soft/Python310/Scripts:$PATH"
cd /c/Users/32110/Desktop/a-liner-main/sample_data/a_Stx-phages
bash runme.sh
```

输出：

```text
Stx-prophage-regions.pdf
Stx-prophage-regions.log
```

该示例包含：

- Excel 序列配置；
- BLASTN 比对；
- GenBank 和 GFF3 注释；
- CDS 和 pseudogene 箭头；
- 基因颜色图例；
- GC 含量散点；
- 区域高亮。

### 12.2 示例二：鸵鸟和鸸鹋性染色体

```bash
export PATH="/d/soft/Python310/Scripts:$PATH"
cd /c/Users/32110/Desktop/a-liner-main/sample_data/b_ostrich-emu
bash runme.sh
```

输出：

```text
ostrich-emu_sex-chromosomes.pdf
log_ostrich-emu_sex-chromosomes
```

该示例包含：

- TSV 序列配置；
- minimap2 PAF 比对；
- 染色体尺度绘图；
- 拷贝数散点；
- 序列轨道和散点背景高亮；
- 一致性颜色图例。

### 12.3 从 PowerShell 调用 Git Bash

如果 `bash` 已加入 PATH：

```powershell
Set-Location 'C:\Users\32110\Desktop\a-liner-main\sample_data\a_Stx-phages'
bash runme.sh
```

如果提示找不到 `a-liner`，在 Git Bash 中先设置：

```bash
export PATH="/d/soft/Python310/Scripts:$PATH"
```

## 13. 常用参数

### 13.1 输入和输出

| 参数 | 作用 | 默认值 |
|---|---|---|
| `-i FILE` | TSV 序列配置 | 必填选项之一 |
| `--xlsx FILE` | Excel 序列配置 | 必填选项之一 |
| `--xlsx_sheet NAME` | Excel 工作表 | 第一个工作表 |
| `--out PREFIX` | 输出文件前缀 | `out` |
| `--figure_size W H` | PDF 宽高，单位为英寸 | `6 0`，高度自动 |

`-i` 和 `--xlsx` 只能选择一个。

### 13.2 序列布局

| 参数 | 作用 | 默认值 |
|---|---|---|
| `--seq_layout` | `left`、`center` 或 `right` | `left` |
| `--margin_bw_seqs` | 同一轨道中序列间距 | 自动 |
| `--xlim_max` | 横轴最大绘图范围 | 自动 |
| `--left_margin` | 图像左边距，范围 0.05～0.50 | 自动 |
| `--seq_color` | 序列线颜色 | `grey` |
| `--seq_font_size` | 序列名称字号 | `6` |
| `--seq_thickness` | 序列线宽 | `1.5` |

### 13.3 比例尺

| 参数 | 作用 | 默认值 |
|---|---|---|
| `--scale legend` | 只显示独立比例尺 | 默认 |
| `--scale tick` | 每条序列显示坐标刻度 |  |
| `--scale both` | 同时显示比例尺和刻度 |  |
| `--tick_width` | 刻度间隔，单位 bp | 自动 |
| `--tick_font_size` | 刻度字号 | `3` |

### 13.4 比对

| 参数 | 作用 | 默认值 |
|---|---|---|
| `-a` / `--alignment` | 自定义七列比对文件 | 无 |
| `--blastn` | BLASTN outfmt 6 | 无 |
| `--lastz` | LASTZ general 格式 | 无 |
| `--mummer` | MUMmer show-coords | 无 |
| `--minimap2` | minimap2 PAF | 无 |
| `--min_identity` | 最低一致性百分比 | `70` |
| `--min_alignment_len` | 最短比对长度 | `0` |
| `--alignment_alpha` | 比对色块透明度，0～1 | `0.5` |
| `--colormap` | 一致性色带编号 | `5` |
| `--include_nonadjacent` | 显示非相邻轨道比对 | 关闭 |

颜色方案：

| 编号 | Matplotlib 色带 |
|---|---|
| `0` | `bone_r` |
| `1` | `hot_r` |
| `2` | `BuPu` |
| `3` | `YlOrRd` |
| `4` | `YlGnBu` |
| `5` | 原始彩虹色带 |

### 13.5 基因

| 参数 | 作用 | 默认值 |
|---|---|---|
| `--gff3` | GFF3 文件 | 无 |
| `--gff_xlsx` | GFF 风格 Excel | 无 |
| `--gb` | GenBank 文件 | 无 |
| `--feature` | 要显示的 feature 类型 | `gene` |
| `--gene_thickness` | 基因箭头相对厚度 | `3` |
| `--gene_label_attr` | 基因标签属性 | `gene` |
| `--gene_font_size` | 基因名称字号 | `3` |
| `--gene_font_rotation` | 基因名称旋转角度 | `75` |
| `--gene_color` | 基因填充颜色 | `white` |
| `--gene_edge_color` | 基因边框颜色 | 无 |
| `--feature_color_map` | 基因关键词颜色表 | 无 |

### 13.6 高亮和散点

| 参数 | 作用 | 默认值 |
|---|---|---|
| `--highlight` | 序列高亮文件 | 无 |
| `--h_alpha` | 序列高亮透明度 | `0.3` |
| `--h_thickness` | 序列高亮厚度 | `3.5` |
| `--scatter` | 散点文件 | 无 |
| `--marker_color` | 散点颜色 | `deeppink` |
| `--marker_size` | 散点大小 | `3` |
| `--marker_style` | 散点形状 | `.` |
| `--scatter_space` | 散点轨道相对高度 | `0.8` |
| `--scatter_min` | 散点纵轴最小值 | `0` |
| `--scatter_max` | 散点纵轴最大值 | `4` |
| `--scatter_ylines` | 水平参考线 | 无 |
| `--background_color` | 散点背景颜色 | `whitesmoke` |
| `--sp_highlight` | 散点背景高亮文件 | 无 |
| `--sp_h_alpha` | 散点背景高亮透明度 | `0.3` |

查看解释器中的完整参数：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' --help
```

## 14. 从 Python 代码调用

可以将 `a-liner` 集成到自己的 Python 脚本。

创建 `run_a_liner.py`：

```python
from a_liner.cli import run
from a_liner.common import get_args

args = get_args(
    [
        "-i",
        "sequence_config.txt",
        "--blastn",
        "blastn.txt",
        "--gff3",
        "annotation.gff3",
        "--feature",
        "CDS",
        "--out",
        "python_result",
    ]
)

run(args)
```

运行：

```powershell
& 'D:\soft\Python310\python.exe' '.\run_a_liner.py'
```

输出：

```text
python_result.pdf
```

不推荐使用下面的命令启动当前版本：

```powershell
& 'D:\soft\Python310\python.exe' -m a_liner.cli
```

当前 `cli.py` 没有直接调用 `main()` 的模块入口。请使用已经安装的 `a-liner.exe`，或者按照上面的 Python API 方式调用。

## 15. PowerShell 使用注意事项

### 15.1 路径包含空格

始终使用单引号：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'D:\My Project\sequence config.txt' `
  --out 'D:\My Project\result'
```

### 15.2 十六进制颜色

颜色必须加引号，否则在某些 Shell 中 `#` 会被当作注释：

```powershell
--marker_color '#C33399'
```

### 15.3 PowerShell 通配符

PowerShell 调用原生命令时，不一定会像 Bash 那样展开 `*.gff`。

最稳妥的方式是明确列出文件：

```powershell
--gff3 'sample1.gff' 'sample2.gff'
```

或者先收集文件：

```powershell
$gffFiles = Get-ChildItem '.\input\*.gff' | ForEach-Object FullName

& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --gff3 $gffFiles `
  --out 'result'
```

### 15.4 PowerShell 换行符

PowerShell 使用反引号续行：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --out 'result'
```

反引号后面不能再有空格。

## 16. 常见错误

### 16.1 找不到 `a-liner`

错误：

```text
a-liner: command not found
```

PowerShell 中使用完整路径：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' --help
```

或者重新安装：

```powershell
Set-Location 'C:\Users\32110\Desktop\a-liner-main'
& 'D:\soft\Python310\python.exe' -m pip install -e .
```

### 16.2 输入文件不存在

先确认路径：

```powershell
Test-Path 'sequence_config.txt'
```

建议使用绝对路径排查：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'C:\data\sequence_config.txt' `
  --out 'C:\data\result'
```

### 16.3 找不到序列或比对不显示

检查以下 ID 是否完全一致：

- 序列配置的 `ID`；
- BLASTN 的 `qseqid` 和 `sseqid`；
- PAF 的 query 和 target 名称；
- GFF3 第一列；
- GenBank `record.id`；
- 高亮文件第一列；
- 散点文件第一列。

ID 匹配区分字符，版本号也属于 ID 的一部分。例如：

```text
NC_000001
```

与：

```text
NC_000001.11
```

并不相同。

### 16.4 比对被过滤

检查：

```text
--min_identity
--min_alignment_len
```

可以先降低过滤条件：

```powershell
--min_identity 70 --min_alignment_len 0
```

当前版本不要设置：

```text
--min_identity 100
```

当存在 100% 一致性比对时，当前颜色换算可能发生除零错误。需要严格筛选 100% 比对时，建议暂时使用：

```text
--min_identity 99
```

### 16.5 散点绘图发生除零错误

必须满足：

```text
scatter_max > scatter_min
```

不要写：

```powershell
--scatter_min 1 --scatter_max 1
```

正确示例：

```powershell
--scatter_min 0 --scatter_max 3
```

### 16.6 图像尺寸错误

`--figure_size` 必须使用正数，或者使用 `0` 触发自动值：

```powershell
--figure_size 6 0
```

不要使用负数：

```powershell
--figure_size -1 -1
```

### 16.7 颜色无效

可以使用：

```text
black
white
red
deeppink
#FF0000
#66CCFF
```

不确定颜色是否合法时，可以在 Python 中检查：

```powershell
& 'D:\soft\Python310\python.exe' -c "from matplotlib.colors import is_color_like; print(is_color_like('#66CCFF'))"
```

输出 `True` 表示合法。

### 16.8 GFF3 基因不显示

依次检查：

1. GFF3 第一列是否匹配序列 `ID`；
2. 基因坐标是否完整落在配置文件的 `start～end` 范围中；
3. `--feature` 是否包含目标类型；
4. GFF3 是否是九列或十列 Tab 分隔格式；
5. `--gene_label_attr` 是否与 attributes 中的键一致。

例如 GFF3：

```text
chr1	source	CDS	100	900	.	+	0	ID=cds1;gene=abcA
```

对应参数：

```text
--feature CDS --gene_label_attr gene
```

## 17. 输出日志和结果检查

程序运行时会在标准输出中打印实际使用的参数，最后出现：

```text
result.pdf has been generated.
```

在 PowerShell 中保存日志：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --out 'result' `
  *> 'result.log'
```

检查 PDF：

```powershell
Get-Item '.\result.pdf'
```

建议每次确认：

- PDF 文件存在；
- PDF 文件大小不是 0；
- 序列名称正确；
- 比对连接的是预期轨道；
- 正负链方向正确；
- 一致性图例范围正确；
- 基因箭头方向正确；
- 散点纵轴范围正确。

## 18. 推荐工作流程

不要一开始就同时加载全部数据。建议逐步增加图层：

### 第一步：只画序列

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --out 'step1_sequences'
```

确认轨道顺序、名称、范围和方向。

### 第二步：加入比对

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --blastn 'blastn.txt' `
  --out 'step2_alignment'
```

确认 ID、坐标和方向。

### 第三步：加入基因

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --blastn 'blastn.txt' `
  --gff3 'annotation.gff3' `
  --feature CDS `
  --out 'step3_genes'
```

### 第四步：加入高亮和散点

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' `
  -i 'sequence_config.txt' `
  --blastn 'blastn.txt' `
  --gff3 'annotation.gff3' `
  --feature CDS `
  --highlight 'highlights.txt' `
  --scatter 'gc_content.txt' `
  --scatter_min 30 `
  --scatter_max 70 `
  --out 'step4_complete'
```

这种逐层检查的方法最容易发现 ID、坐标或文件格式问题。

## 19. 快速命令清单

检查程序：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' --help
```

只画序列：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' -i 'sequence_config.txt' --out 'result'
```

BLASTN：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' -i 'sequence_config.txt' --blastn 'blastn.txt' --out 'result'
```

minimap2：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' -i 'sequence_config.txt' --minimap2 'alignment.paf' --out 'result'
```

GFF3：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' -i 'sequence_config.txt' --gff3 'annotation.gff3' --feature CDS --out 'result'
```

完整帮助：

```powershell
& 'D:\soft\Python310\Scripts\a-liner.exe' --help
```

官方英文说明见项目根目录的 `README.md`。

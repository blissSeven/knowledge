# DNA 基序发现：从概率模型到可检验的机制性表述

**栏目**：Review（综述） · **语言**：中文 · **风格**：Nature

> 体裁说明：本文为基于公开文献的综述稿件，不含作者本人的原始数据、实验或统计。所有事实性主张均引用已发表文献；涉及版面示意图处标注为 schematic（概念示意），不含实测数值。参考文献链接均经核实。

---

## 摘要

转录因子通过识别基因组中的短序列基序来编排基因表达，因此定位这些基序是解读调控语法的前提。三十余年来，基序发现的方法经历了从枚举共识串、概率模型到深度学习的多次更迭，但一个根本张力始终未解：越是能够刻画真实调控语法的表示，往往越难被高效求解。本文梳理这一领域的方法谱系与关键转折，指出组合优化时代积累的期望最大化（expectation maximization, EM）与吉布斯采样框架、基准评测揭示的能力边界，以及深度学习带来的表示跃迁，共同把问题推向新的层面。我们据此认为，领域正在从"找出一个矩阵"转向"找回一个可检验的机制性表述"；基序语法、因果验证、评测方法学与 DNA 语言模型的分词策略，将决定这一转向能否兑现。

---

## 引言

人类基因组中，蛋白质编码序列不足百分之二，而调控元件散布于其余广阔的非编码区。转录因子（transcription factor, TF）与这些元件的结合，主要由一段约 6–20 个碱基的短序列所决定，这段序列被称为基序（motif）。找出基序，就等于拿到了解读调控语法的钥匙：它解释了细胞类型特异的表达从何而来，也为解释非编码区变异的效应提供了坐标[1,2]。

问题看似简单，实则顽固。给定一组来自染色质免疫沉淀测序（ChIP-seq）等实验的序列，希望从中找出过代表的保守模式。生物学家在二十世纪九十年代就为此写出了一批算法，但三十年后，这个"从序列里找模式"的问题仍未真正解决[1,3]。原因不在于缺少算力，而在于一个反复出现的取舍：简单表示可以被穷举、可以证明最优，却表达力贫乏；而贴近生物现实的表示——碱基间的依赖、基序的组合与间距、细胞类型特异的语法——又会让搜索空间膨胀到难以求解。

本文沿这条"表示力与可解性"的主线组织。我们先看表示法的演化，再看与之配对的优化器，随后讨论一次改变领域自我评价的基准评测，继而进入深度学习带来的范式替换，最后落脚于尚待解决的方向。全文的论点是：基序发现的目标正在迁移——从产出一个可数的权重矩阵，转向产出一个可被实验检验的机制性表述。

---

## 表示法是骨架

基序发现的方法分类，首先取决于如何表示一个基序，而非用什么优化器去搜索它[1,3]。

**确定性表示**。最早的方法把基序写成共识串、IUPAC 正则式或允许若干错配的字符串。这类表示可以被枚举、可以证明找到最优解，代价是只适合短而强的模式，且无法表达碱基位置的连续变化。当真实结合位点分散在多个相似序列上时，共识串很快失效。

**概率表示**。位置权重矩阵（position weight matrix, PWM）用一张 4×l 的矩阵描述每个位置上四种碱基的偏好，配合背景模型计算似然。PWM 在统计上可解释、在工程上易实现，成为过去三十年的绝对主流[1,2]。几乎所有组合优化技术——贪心、局部搜索、模拟退火、遗传算法——都曾套用在 PWM 上。PWM 的代价是位置独立假设：它默认一个位置上碱基的贡献与其他位置无关，而真实的结合位点并非如此。

**高阶依赖模型**。为突破位置独立假设，领域发展出二核苷酸/三核苷酸矩阵、贝叶斯网络、隐马尔可夫模型与变阶模型。其中 BaMM 用贝叶斯方法训练高阶依赖，在报告的大规模人类 ChIP-seq 基准上，较 PWM 取得了可观的精度提升[7]。这类表示表达力更强，却把可解性推到极限：高阶参数的估计需要更多数据，也更易陷入局部最优。也有一些工作绕开 PWM 另辟路径，例如用带间隙 k-mer 核直接刻画序列的 gkmSVM[8]，以及基于隐马尔可夫模型采样的 NestedMICA[11]。

表示法的演化史，本质上是一部不断用可解性换取表达力的历史。每一次"更真实"的表示，都让搜索变得更难。

---

## 优化器是血肉

在 PWM 这一主流表示下，两条最经典的优化路线常被放进同一个极大似然框架中比较[12]。

**期望最大化是确定性版本**。MEME 以真实子串作为起点迭代更新矩阵，取消了"每条序列都必须含一个 motif"的早期假设，并在找到一个 motif 后以概率方式擦除它，因而能够发现多个基序[3,5]。EM 收敛稳定，但对初值敏感，容易停在局部极大。

**吉布斯采样是随机版本**。它按权重对子序列抽样并反复重建矩阵，对初值不敏感，却更依赖"每条序列都含 motif"的假设[4]。当输入序列中混有不含结合位点的片段时，它的表现会下降。

两者的分歧并不只是实现细节，而是反映了对数据的不同信念：EM 更信任数据的整体分布，吉布斯采样更信任"每条序列都有故事"的结构假设。理解这一点，比记住某个工具的默认参数更有用——它决定了在给定的实验设计下，该信任哪一类方法。

> **Figure 1（schematic）**：表示力—可解性权衡示意。横轴为表示的表达力（共识串→PWM→高阶依赖→深度表示），纵轴为可求解性，曲线单调下降；标注 MEME、Gibbs、BaMM、BPNet 所处位置。图注需说明：纵轴为概念刻度，非实测值。

---

## 一次祛魅：基准评测的教训

如果只看单个工具的论文，领域似乎在稳步进步。2005 年的一项大规模评测改变了这一印象：Tompa 等比较了 13 种工具，发现没有任何一种全面占优，不同工具的结果重叠很小，而且所有工具在真实数据上的表现都远逊于合成数据[6]。

这个结论至今仍是领域最实用的经验之一：**组合少量互补的工具，关注排名靠前的一组候选（top-N），而不是迷信单一工具的第一名**。随后，基准方法本身也被反思——需要区分"算法不行"与"表示力不够"两种失败，否则一次评测无法告诉我们问题出在哪里[1]。

祛魅的意义在于，它把基序发现从"寻找最好的算法"重新定义为"理解失败的来源"。这一转向，恰好为表示力更强但难以用传统指标衡量的方法铺平了道路。

---

## 深度学习的范式替换

2015 年前后，领域发生了一次范式替换：从"直接搜索基序"，转向"先学习序列到功能的映射，再从中反解基序"[26]。

DeepBind 开创了用卷积神经网络（CNN）预测 DNA—蛋白结合的先河[13]；DeepSEA 与 Basset 把这一思路扩展到非编码变异效应与染色质可及性[14,15]。此后，BPNet 用扩张卷积在碱基分辨率上同时预测信号计数与形状，展现出对基序组合与间距的敏感性——所谓"soft motif syntax"[16]。Enformer 与 Borzoi 则把感受野扩展到十万碱基量级，以建模长程调控[17,18]。

一条重要的工程经验是：纯 Transformer 难以直接处理裸序列，而"CNN 塔 + Transformer"的混合架构更为可行[25,26]。这提示，架构选择必须尊重序列数据的局部性先验，而不能简单照搬自然语言处理的成功配方。

深度学习的代价是可解释性从"内建"变成了"事后归因"。PWM 天然是一个可读的矩阵，而一个深层的卷积网络不是。于是，如何把模型学到的东西还原成生物学家能理解的基序，成为新的核心问题。

> **Figure 2（schematic）**：两大范式对比。左栏"生成式搜索"：候选基序→打分→迭代优化→PWM；右栏"判别式学习"：序列→深度模型→功能预测→归因→基序。中间以箭头连接，标注"表示力↑、可解释性↓"。图注需说明：为概念框架，非具体模型结构图。

---

## 归因反解与基序语法

TF-MoDISco 通过聚类碱基级重要性片段（seqlet）输出非冗余的 PWM，已成为事实上的标准工具[21]。但它的局限同样明显：速度慢、结果冗余，且不揭示基序之间的组合与间距关系。F-MoDA 等方法尝试用傅里叶变换与层次聚类来改善这一环节[22]。

真正困难的是基序语法（motif grammar）：基序的数量、间距、方向、协同与竞争如何编码进调控输出？PWM 表达不了它，深度网络能学到它，却难以把它干净地反解出来[26]。一些轻量级、可解释的架构开始尝试显式编码距离依赖[23]；另一条路径则把蛋白语言模型的潜空间稀疏化，以直接读出可解释特征[24]。

这两条路径看似相反——一条回到精简的显式模型，一条走向高维潜空间的特征解耦——实则共享同一个目标：让基序的表述重新变得可检验。

---

## 展望

领域正在跨越一道分水岭：目标不再是"找到一张矩阵"，而是"找到一个可检验的机制性表述"。以下几个方向将决定这一转向能否兑现。

**其一，基序语法的显式建模。** 让模型同时表达碱基偏好与基序间的空间关系，并在保持可解释性的前提下接受实验检验，是 PWM 之后最实质的表示升级[23,26]。

**其二，从相关性走向因果。** 模型给出的重要性分数不等于因果机制。在存在上位效应时，效应量会被系统性错误标定，因此需要与 MPRA 等通量实验交叉验证，才能把"模型在意"变成"生物依赖"。

**其三，评测方法学本身。** 现有基准难以区分算法能力与表示能力，合成数据带有模型偏置[6,1]。没有可信的尺子，任何进步都无法被确认。这是领域公认的开放难题，也是容易被忽视的基础工作。

**其四，DNA 语言模型的分词策略。** 如何设计生物学相关、能正确切分基序的词元，而非照搬自然语言处理的方案，已被 2025 年的综述列为首要开放问题[25]。分词方式直接决定了模型能否"看见"短而关键的调控模式。

**其五，单细胞与空间维度。** 单细胞 ATAC-seq 数据极端稀疏，带来新的统计挑战；scBasset 等模型虽能在细胞状态间迁移，泛化能力仍显不足[20]。

**其六，RNA 结构基序。** 能同时刻画序列与结构、又包含假结的方法依然缺乏；引入 SHAPE-seq 等实验约束是明显却尚未实现的方向[27]。

把这些方向放在一起看，一个共同的诉求浮现出来： Next-generation 的基序发现，必须是可解释、可验证、可跨尺度迁移的。它不再是一个纯粹的算法问题，而是一个连接计算预测与实验生物学的接口问题。

---

## 实用指南

面对具体任务时，方法选择应服务于目标，而非追随潮流。下表中的建议均基于前述文献的综合，属综述性判断，不含作者本人的基准测试。

| 目标 | 推荐路径 | 依据 |
|---|---|---|
| 快速获得可信候选基序 | MEME-ChIP / HOMER / GimmeMotifs 组合，取 top-N | 单一工具不全面占优，需互补与排序[6,9,10] |
| 细胞类型特异的调控语法 | BPNet / ChromBPNet + 归因 + TF-MoDISco | 碱基分辨率模型捕捉组合与间距[16,19,21] |
| 变异效应预测 | ChromBPNet / Enformer / Borzoi + 饱和突变 + MPRA 验证 | 长程模型预测效应，需实验校准[17,18,19] |
| 蛋白功能位点 | InterProScan / ELM，短 pattern 命中须人工核对上下文 | 短模式假阳性高，依赖上下文[28] |
| RNA 结构基序 | CMfinder 类方法；含假结用专门方法，尽量引入 SHAPE 约束 | 序列—结构联合建模仍不成熟[27] |

一个贯穿始终的忠告：**基序命中只是假设，不是结论**。它必须结合序列上下文、结构域组织、进化保守性与功能证据，才可能被当作生物学解释。

---

## 待补清单（诚信标注）

本稿未包含作者原始数据，以下为按 Nature 规范应由作者在正式投稿前补齐的内容，写作过程中未以任何形式虚构：

- `[待补：作者、单位、通讯作者信息]`
- `[待补：利益冲突声明]`
- `[待补：作者贡献声明]`
- `[待补：数据可得性声明]`
- `[待补：基金资助信息]`
- `[待补引用：如需覆盖 ELM 之外更多蛋白基序数据库的最新版本文献]`
- `[待补：Figure 1–2 的最终绘图与图注定稿，当前为 schematic 文本描述]`

---

## 参考文献

1. Sandve GK, Drabløs F. *A survey of motif discovery methods in an integrated framework.* Biology Direct (2006). https://europepmc.org/articles/PMC1479319/
2. Das D, Dai HK. *A survey of DNA motif finding algorithms.* BMC Bioinformatics (2007). https://pmc.ncbi.nlm.nih.gov/articles/PMC2099490
3. Bailey TL, Elkan C. *Fitting a mixture model by expectation maximization to discover motifs in biopolymers.* (1994). https://pubmed.ncbi.nlm.nih.gov/7584402/
4. Lawrence CE, et al. *Detecting subtle sequence signals: a Gibbs sampling strategy for multiple alignment.* Science (1993). https://pubmed.gov/8211139/
5. Bailey TL, et al. *The MEME Suite.* Nucleic Acids Research (2015). https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4489269/
6. Tompa M, et al. *Assessing computational tools for the discovery of transcription factor binding sites.* Nature Biotechnology (2005). https://pubmed.ncbi.nlm.nih.gov/15637633
7. Siebert S, Söding J. *Bayesian Markov models consistently outperform PWMs at predicting motifs in nucleotide sequences.* Nucleic Acids Research (2016). https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5291271/
8. Ghandi M, et al. *gkm-SVM: a method for predicting regulatory DNA function.* Bioinformatics (2016). https://doi.org/10.1093/bioinformatics/btw203
9. Machanick P, Bailey TL. *MEME-ChIP: motif analysis of large DNA datasets.* Bioinformatics (2011). https://doi.org/10.1093/bioinformatics/btr189
10. van Heeringen SJ, Veenstra GJC. *GimmeMotifs: a de novo motif prediction pipeline.* Bioinformatics (2011). https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3018809
11. Down T, Hubbard T. *NestedMICA: sensitive inference of over-represented motifs.* Nucleic Acids Research (2005). https://www.ncbi.nlm.nih.gov/pubmed/15760844
12. *A unifying view of EM and Gibbs sampling in the maximum-likelihood framework.* (2009). https://www.sciencedirect.com/science/article/pii/S0167865509002335
13. Alipanahi B, et al. *Predicting the sequence specificities of DNA- and RNA-binding proteins by deep learning.* Nature Biotechnology (2015). https://pubmed.ncbi.nlm.nih.gov/26213851/
14. Zhou J, Troyanskaya OG. *Predicting effects of noncoding variants with deep learning-based sequence model.* Nature Methods (2015). https://doi.org/10.1038/nmeth.3547
15. Kelley DR, et al. *Sequential regulatory activity prediction across chromosomes with convolutional neural networks.* Genome Research (2016). https://doi.org/10.1101/gr.200535.115
16. Avsec Ž, et al. *Base-resolution models of transcription-factor binding reveal soft motif syntax.* Nature Genetics (2021). https://pmc.ncbi.nlm.nih.gov/articles/8812996/
17. Avsec Ž, et al. *Effective gene expression prediction from sequence by integrating long-range interactions.* Nature Methods (2021). https://doi.org/10.1038/s41592-021-01252-x
18. Linder J, et al. *Predicting RNA-seq coverage from DNA sequence as a unified model of gene regulation.* Nature Genetics (2025). https://doi.org/10.1038/s41588-024-02053-6
19. Pampari S, et al. *ChromBPNet: bias-corrected base-resolution models of chromatin accessibility.* (2024/2025). https://europepmc.org/article/PMC/PMC11741299
20. Yuan Y, Kelley DR. *scBasset: sequence-based modeling of single-cell chromatin accessibility.* Nature Methods (2022). https://doi.org/10.1038/s41592-022-01562-8
21. Shrikumar A, et al. *TF-MoDISco: identification of sequence motifs from deep learning models of DNA sequence function.* (2018). https://ar5iv.labs.arxiv.org/html/1811.00416
22. Yaish O, Orenstein Y. *F-MoDA: Fourier-based motif discovery from attributions.* PMLR v311 (2025). https://proceedings.mlr.press/v311/yaish25a.html
23. Hu S, et al. *InfluenceNet: interpretable modeling of motif interactions.* bioRxiv (2025). https://doi.org/10.1101/2025.11.20.689553
24. Hou Y, et al. *MotifAE: sparse interpretable features from protein language models.* bioRxiv (2025). https://doi.org/10.1101/2025.11.04.686576
25. Testagrose A, Boucher C. *Tokenization and deep learning architectures in genomics.* (2025). https://pmc.ncbi.nlm.nih.gov/articles/PMC12356405/pdf/main.pdf
26. *Deep learning sequence models for transcriptional regulation.* Annual Review of Genomics and Human Genetics (2025). https://doi.org/10.1146/annurev-genom-021623-024727
27. *A survey on algorithms to characterize transcription factor binding sites.* (2023). https://pmc.ncbi.nlm.nih.gov/articles/PMC10422928/pdf/bbad156.pdf
28. *The Eukaryotic Linear Motif resource: 2024 update.* https://europepmc.org/articles/PMC10767929/

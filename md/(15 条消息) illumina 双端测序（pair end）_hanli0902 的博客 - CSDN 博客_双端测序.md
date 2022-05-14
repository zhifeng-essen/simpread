> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/hanli1992/article/details/82980042)

本文来自 sixu_9days 的 CSDN 博客 ：[https://blog.csdn.net/sixu_9days/article/details/78948914](https://blog.csdn.net/sixu_9days/article/details/78948914)

illumina 测序的核心在于利用可逆终止的、荧[光标](https://so.csdn.net/so/search?q=%E5%85%89%E6%A0%87&spm=1001.2101.3001.7020)记的 dNTP 进行边合成边测序（Sequencing-By-Synthesis,_SBS_）

Flowcell（流动池）是有着 2 个或 8 个 lane（泳道）的玻璃板，每个 lane 可以测一个样本或者多样本的混合物，且随机布满了能够与文库两端接头分别**互补配对或一致**的寡核苷酸（oligos，P7 和 P5 接头）。一个 lane 包含两列，每一列有 60 个 tile，每个 tile 会种下不同的 cluster，每个 tile 在一次循环中会拍照 4 次（每个碱基一次）。

![](https://img-blog.csdn.net/2018100911424515?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbmxpMTk5Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

一、Library Preparation 文库的构建
===========================

1. 利用转座子（transposome）对双链 DNA 进行剪切以及接头（adapter）的连接

![](https://img-blog.csdn.net/20181009130602911?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbmxpMTk5Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![](https://img-blog.csdn.net/2018100913060624?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbmxpMTk5Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2. 接头连接成功后，利用低循环扩增技术在接头处进行修饰，分别在两端添加 sequencing primer binding site1 / sequencing primer binding site2（即测序引物结合位点）、index1/index2 以及我们称之 P5 和 P7 的寡核苷酸序列

![](https://img-blog.csdn.net/20180102102643081)

下图是维基百科的示意图，详细一些。

![](https://img-blog.csdn.net/20180102102955710?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**注意：**

1.  P5 和 P7 是不同的，它们分别和 flowcell 上的接头互补和相同。为了方便阐述，将与 P5 互补的接头称为 P5’，与 P7 互补的接头称为 P7’。
2.  index1 和 index2 也是不同的，与 P5 相连的是 index2，与 P7 相连的是 index1

关于 index，也叫 barcodes，因为一个 lane 可以同时测多个样品，为了避免混淆样品的 read products，每种样品的 DNA 由一种 index 修饰，这样测序得到的 reads 都是具有 index 标记的，在测序结果中，依据之前标签与样品的对应关系，就可以获得对应样品的数据。而这里的 **index1 和 index2 是为了区分 paired-end 测序得到的双端 reads**。

二、Cluster generation 簇生成
========================

1. Flowcell 上随机分布了两种不同的寡核苷酸序列，分别**与 P5 互补（即 P5’），与 P7 一致（即 P7）**。

![](https://img-blog.csdn.net/20180102103623636?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2. 待测 sequence 通过 P5 与 folwcell 上的 P5’序列杂交互补，以待测 sequence 为模板进行互补链（即 reverse strand）的延伸，互补链的两端为 P5’和 P7’。

![](https://img-blog.csdn.net/20180102104221274?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102143647077?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102103954806?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

3.  接下来模板链被切断并洗下

![](https://img-blog.csdn.net/20180102104507815?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

Reverse strand 的 P7’与 Flowcell 上的 P7 杂交互补，进行链的合成，这就是我们所熟知的**桥式 PCR**

![](https://img-blog.csdn.net/20180102104958854?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102143647077?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102105105395?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

接下来合成的双链被解链，再分别与 Flowcell 上的接头杂交互补，延伸，解链，杂交，延伸，解链... 如此重复 35 个循环

![](https://img-blog.csdn.net/20180102105631447?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102143647077?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102110221085?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![](https://img-blog.csdn.net/20180102110315405?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

4. 桥式 PCR 完成后，使用 NAOH 将双链解链，并利用甲酰胺基嘧啶糖苷酶（Fpg）对 8 - 氧鸟嘌呤糖苷（8-oxo-G）的选择性切断作用，选择性地将 P5’与链的连接切断，**留下与 Flowcell 上 P7 连接的链**，也就是 Forward strand。同时游离的 3’端被阻断，防止不必要的 DNA 延伸

![](https://img-blog.csdn.net/20180102110455607?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

三、测序
====

1. 测序引物（sequencing primer）结合到靠近 P5 的测序引物结合位点 1（sequencing primer binding site 1）上，在系统中加入四种 dNTP 和 DNA 聚合酶。这里的 dNTP 有两个特点：它是有荧光基团标记的，每种碱基标记的荧光基团不一样；它的 3’末端连了一个叠氮基，这个叠氮基能够阻断后面的碱基与它相连

![](https://img-blog.csdn.net/20180102110709747?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

因此在聚合酶的作用下，与 Forward strand 相应位置碱基配对的 dNTP 就会结合到新合成的链上，而由于叠氮基的存在，后面的 dNTP 无法继续连接。这时用水将剩余的 dNTP 和酶给冲掉，将 Flowcell 进行扫描，扫描出来的荧光对应的碱基的配对碱基即是该链该位置的碱基。同时在这个 Flowcell 上有成千上万个 cluster 也在进行同样的反应，因此一个循环就能同时检测多个样本（这也是高通量的核心所在）。这个循环完成后，加入化学试剂把叠氮基和标记的荧光基团切掉，进行下一个循环（碱基的连接、检测与切除）。如此重复直至所有链的碱基序列被检测出，也就是 Forward read 序列。

![](https://img-blog.csdn.net/20180102110801150?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2. Index 测序：所有循环结束后，read products 被洗掉，index1 primer 与链上 index primer1 结合位点杂交配对，进行 index1 的合成及检测

![](https://img-blog.csdn.net/20180102111207627?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

3. Index1 测序完成后，洗脱测序产物。此时机器已通过荧光得到了 index1 的序列

![](https://img-blog.csdn.net/20180102111345135?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

4.Index2 测序：Forward strand 顶端的 P5 序列与 Flowcell 上的 P5’杂交配对，进行 index2 测序。测序完成后洗脱产物

![](https://img-blog.csdn.net/20180102121740290?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102143647077?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102121753750?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

四、Paried-end sequencing（即对 Reverse strand 测序）
=============================================

1. 洗脱 index2 测序产物后，以 Flowcell 上的 P5’为引物，Forward strand 为模板进行桥式扩增，得到双链

![](https://img-blog.csdn.net/20180102122603418?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2. NAOH 使双链变性为单链，并洗去已经测序完成的 Forward strand

![](https://img-blog.csdn.net/20180102122804843?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102143647077?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)![](https://img-blog.csdn.net/20180102122822016?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

3.   类似的，readprimer2 结合到靠近 P7’的 read primer binding site 2 开始对 Reverse strand 的测序。测序完成后即可得到 Reverse read 序列。

![](https://img-blog.csdn.net/20180102123012562?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2l4dV85ZGF5cw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

前面介绍的都是 paired-end 的测序，而 single-end 测序方式是只将 index，sequencing primer binding site 以及 P7/P5 添加到 fragamented DNA 片段的一端，另一端直接连上 P5/P7，将片段固定在 Flowcell 上桥式 PCR 生成 DNA 簇，然后单端测序读取序列

illumina 的官方视频：

[http://v.youku.com/v_show/id_XMTI1MjA5Mzg5Mg==.html?spm=a2h0k.8191407.0.0&from=s1.8-1-1.2](http://v.youku.com/v_show/id_XMTI1MjA5Mzg5Mg==.html?spm=a2h0k.8191407.0.0&from=s1.8-1-1.2)
---
layout: post
title: Tempered Adversarial Networks
---

## TL;DR
* GANの学習は安定しない
  * GeneratorとDiscriminatorのバランスを調整するのが難しい
  * Mode Collapse
* 普通のGANはfixされたデータ分布に生成分布を近づける
* この手法ではデータ分布を生成分布に近づけるようにして安定化を図った
  * データ分布を生成分布に近づけるために Lens Networkを用いる
  * この枠組みは，DCGAN・LSGAN・WGANなどObjectiveにかかわらず有効

## Tempered Adversarial Networks
GANのもともとのObjectiveは以下  

$$ \min_G\max_DV(D,G)=E_{x\sim{X}}[log(D(x))]+E_{z\sim{Z}}[1-log(D(G(z)))]$$  

データ分布$X$は学習中ずっと固定されたままである．それに対して，生成分布は学習中常に変化している．  
この非対称を解決するためにLens Network ($L$)を用いる．  
Lens を $x\sim{X}$ をDiscriminatorに入力する直前に適用する．

<center><img src="{{ site.baseurl }}/images/post/2018-06-13-Tempered Adversarial Networks/architecture.png" /></center>

<center><img src="{{ site.baseurl }}/images/post/2018-06-13-Tempered Adversarial Networks/models.png" /></center>

Lens の最適化には$L^A_L$と$L^R_L$の2つのLossを用いる．
$L^A_L$は敵対的誤差であり，$L^R_L$は再構成誤差である．

$$ L^A_L=-E_{x\sim{X}}[log(D(L(x)))]$$  

$$ L^R_L=E_{x\sim{X}}\|x-L(x)\|^2_2 $$  

$$ L_L=\lambda L^A_L+L^R_L $$

再構成誤差はLensがすべて0や写像するなどの自明解への収束を防ぐためである．
レンズは再構成とGANの元々のObjectiveのバランスを自動的に取らせている．  
パラメータ$\lambda$は，学習が進むにつれて減少するように設計している．  

$$ \lambda = \left\{
\begin{array}{ll}
1-sin(\frac{t\pi}{2K}) & (t \geq K)\\
0 & (otherwise)
\end{array}
\right. $$  

<center><img src="{{ site.baseurl }}/images/post/2018-06-13-Tempered Adversarial Networks/lambda_schedule.png" /></center>

$\lambda=0$になると，Lensは取り払ってしまう．  
論文内では$K=10^5$で固定している．  
Lensは単射（1対1対応の写像）である必要はない．  
だからLensによって同じデータポイントへ写像されることがあるが，これは分布の複雑さを緩和してくれる．  
このようなLensをDiscriminatorの前につけることで，データ分布を生成分布に近づける．  
それにより学習が安定する．  
以前から，ガウスノイズなどを$x\sim{X}$に付加するテクニックは存在したが，手動でノイズの強さを減衰させていた．  
この手法では，Lensがその調節すらも学習している．  


<center><img src="{{ site.baseurl }}/images/post/2018-06-13-Tempered Adversarial Networks/res_mnist.png" /></center>

<center><img src="{{ site.baseurl }}/images/post/2018-06-13-Tempered Adversarial Networks/fid_mnist.png" /></center>

<center><img src="{{ site.baseurl }}/images/post/2018-06-13-Tempered Adversarial Networks/res_celeba.png" /></center>

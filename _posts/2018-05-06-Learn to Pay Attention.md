---
layout: post
title: Learn to Pay Attention
---

## TL;DR
* 目標はSaliency Mapを使ってCNNが分類を行うときに使う有効な視覚的情報の空間的なサポートを見つけ出し，利用すること
* Saliency Mapを用いることで有効な領域の情報を重視し，無関係な情報を抑制する
* Local feature vector (CNNの中間層の出力)とGlobal feature vector (CNNの後段のFCの出力)を組み合わせる
* 適合度によって重要なLocal feature vectorだけを分類に活用する

## 構造
<img src="{{ site.baseurl }}/images/post/2018-05-06-Learn to Pay Attention/fig.png" />

$S$個のAttention Moduleを持っている．  
$s$個目のAttention Moduleは，長さ$M$のベクトル$N$個からなる集合である．  
ここで，ベクトルの長さ$M$はFeature Mapのチャネル数に等しく，ベクトルの個数$N$はFeature Mapの画素数に等しい．  

$$ \bold{L^s} = \{ \bold{l_1^s}, \bold{l_2^s}, ..., \bold{l_N^s} \} $$  

$L^s$のshapeは，$(BS, N, M)$  


各ベクトルの長さをFC-1の出力$\bold{g}$の長さ$M'$に揃える  

$$ \bold{\hat{l^s_i}} = w\cdot{\bold{l_i^s}} $$  

$\hat{L^s}$のshapeは，$(BS, N, M')$  

Compatibility scoresを求める  

$$ C^s(\bold{\hat{L_s}}, \bold{g}) = \{c_1^s, c_2^s, ..., c_n^s\} $$  

$$ c_i^s =  \bold{\hat{l^s_i}} \cdot{\bold{g}} $$  

$C^s(\bold{\hat{L_s}}, \bold{g})$のshapeは，$(BS, N)$  
Compatibility scoresをSoftmaxで正規化する  

$$ a_i^s =  \frac{exp(c_i^s)}{\sum_j^N exp(c_j^s)} $$  

各Moduleの出力$\bold{g^s}$は  

$$ \bold{g^s} = \sum_i^n a_i^s \cdot{\bold{l_i^s}} $$  

最終的には，全Moduleの出力を連結することでModule全体の出力として，最後にFC層  

$$ \bold{g_{all}} = \{ \bold{g_{1}}, \bold{g_{2}}, ..., \bold{g_{S}}\} $$  

$$ Output = W_{FC2} \cdot{\bold{g_{all}}} $$  
  
<img src="{{ site.baseurl }}/images/post/2018-05-06-Learn to Pay Attention/fig1.png" />

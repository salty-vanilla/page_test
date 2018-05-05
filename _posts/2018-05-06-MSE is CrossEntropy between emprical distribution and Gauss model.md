---
layout: post
title: MSE is CrossEntropy between emprical distribution and Gauss model
---

## MSE (Mean Squared Error)
$$ MSE(y_{true}, y_{pred}) = \frac{1}{N}\sum_{i=1}^{N}|y_{pred}-y_{true}|_2^2 $$

## 最尤推定
最尤推定は経験分布$p_{data}$とモデル分布$p_{model}$のKLDを最小化することとみなせる
$$ D_{KL}(p_{data}| p_{model}) =  E_{x\sim p_{data}}[log[p_{data}(x)]-log[p_{model}(x)]]$$

左項はデータの生成過程にあたる．モデルの関数には関わりを持たないため，以下だけ最小化すればよい

$$ -E_{x\sim p_{data}}[log(p_{model}(x)]$$

## ガウスモデル
ガウスモデルでは，2つのパラメータを持つ
* 平均$p(x)$
* 分散$\sigma^2 =\sum_{i=1}^{N}(x_i-p(x_i))^2$

$$ N(x;p(x),\sigma^2) = \frac{1}{\sqrt{2\pi\sigma^2}}exp(-\frac{(x-p(x))^2}{2\sigma^2}) $$

## 「MSEは経験分布とガウスモデル間のクロスエントロピーである」
$P_{model} = N(x;p(x),\sigma^2)$とすると
$$\begin{eqnarray}
-E_{x\sim p_{data}}[log(p_{model}(x)]
&=& -\frac{1}{N}\sum_{i=1}^{N}log[\frac{1}{\sqrt{2\pi\sigma^2}}exp(-\frac{(x-p(x))^2}{2\sigma^2})] \\
&=& -\frac{1}{N}\sum_{i=1}^{N}log[\frac{1}{\sqrt{2\pi\sigma^2}}]-\frac{(x-p(x))^2}{2\sigma^2} \\
&=& \frac{1}{2N}\sum_{i=1}^{N}\frac{(x-p(x))^2}{\sigma^2}+log[2\pi\sigma^2]
\end{eqnarray}$$

分散$\sigma^2$は，$p(x)$を求めればわかるので，↑の式は

$$ -E_{x\sim p_{data}}[log(p_{model}(x)] \propto sum_{i=1}^{N}(x-p(x))^2 $$

となる．これはMSEのN倍に等しい．

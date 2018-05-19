## TL;DR
* 出力分布を滑らかにすることが目的
* 正則化項 $LDS(x, \theta)$をLossに加算することで滑らかにしている
* Adversarial Trainingと異なるVirtual Adversarial Trainingを提案
* これにより，$x$に対応する正しい$y$を必要としないため，Semisupervisedな学習が可能


## Overview
VAT(Virtual Adversarial Training)は，Adversarial Training同様に摂動に対してロバストにする正則化項を加えることでモデルの汎化性能を上げることができる．  
Adversarial Trainingでは，データポイント$x$の周辺において，モデルのLossを滑らかにすることであった．  
VATでは，データポイント$x$の周辺において，モデルの予測分布$p(y\mid x, \theta)$を滑らかにする．
また，Adversarial Trainingは$x$と$y$のペアが必要であったが．VATでは必要としない．  
そのため，Semisupervisedな学習が可能である．


モデルの予測分布$p(y\mid x, \theta)$を滑らかにするために，$p(y\mid x, \theta)$と$p(y\mid x+r, \theta)$のKLDを最小化するようにLossに加算する正則化項を設計する．($r$は摂動)    
論文内では，KLDは以下のように表記

$$ \Delta_{KL}(r, x, \theta) = KL(p(y\mid x, \theta) \| p(y\mid x+r, \theta))$$

正則化項$LDS(x, \theta)$ (Local Distribution Smoothness)は以下のようになる．

$$ LDS(x, \theta) =  - \Delta_{KL}(r_{v-adv}, x, \theta) $$

$r_{v-adv}$は，$\|r\|_2 \leqq \epsilon$ において，$p(y\mid x, \theta)$を最も変化させるような$r$である．  

$$ r_{v-adv} = \argmax_r \{ \Delta_{KL}(r_{v}, x, \theta \| \|r\|_2 \leqq \epsilon \} $$

正則化項$LDS(x, \theta)$を用いて，目的関数は以下のようになる．  

$$ \frac{1}{N}\sum_{n=1}^N \log p(y_n \mid x_n, \theta ) + \lambda \frac{1}{N}\sum_{n=1}^NLDS(x_n, \theta) $$  

論文内では，$\lambda = 1. $ で固定

## $r_{v-adv}$の算出
本当は，$r_{v-adv}$の算出には，$ \Delta_{KL}(r, x, \theta) $ のHessianを計算する必要がある．  
その理由を以下に示す．  
$\Delta_{KL}(r, x, \theta)$ を $\|r\|_2 \leqq \epsilon$ の条件において最大にするような$r$を求めたい．  
$r=0$ まわりでテイラー展開すると，  

$$\begin{eqnarray} \Delta_{KL}(r, x, \theta) &\simeq& \Delta_{KL}(r=0, x, \theta) + r^T\nabla_r\Delta_{KL}(r=0, x, \theta) + \frac{1}{2}r^T\nabla\nabla_r\Delta_{KL}(r=0, x, \theta)r \\
 &=& \Delta_{KL}(r=0, x, \theta) + r^T\nabla_r\Delta_{KL}(r=0, x, \theta) + \frac{1}{2}r^TH(x, \theta)r \end{eqnarray}$$

KLDの定義から，$r=0$のときには2つの分布は等しくなる．また，必ず正の値となるため，$r=0$のおいて，最小値を取り，$r=0$において勾配は0となる．  

そのため，最大化したいのは

$$ \frac{1}{2}r^TH(x, \theta)r $$

上式は二次形式 (quadratic form)であるため，これを最大化するためには$r$を$H(x, \theta)$の最大固有値の固有ベクトルにすればよい．  
固有値分解は計算的にしんどいので，PowerIterationによって近似計算する．  
この算出アルゴリズムは以下  
<img src='images/pi.png'>

簡単にまとめると以下の3ステップ
* ランダムな単位ベクトル $\bold{d}$を初期化
* $Ip$回，以下の式で$\bold{d}$を更新

$$ \nabla_r\Delta_{KL}(r=\xi d, x, \theta) $$

* 最終に求まった$d$を使って，

$$ r_{v-adv} = \epsilon d$$

これにより，正則化項を求めることができる．  
しかし，1点注意が必要である．
論文内の式12に記述があるように

$$ -\frac{\delta}{\delta \theta}KL(p(y\mid x, \hat{\theta}) \| p(y\mid x+r_{v-adv}, \theta)) $$

これは，$p(y\mid x, \hat{\theta})$を定数とみなすことで，微分したときの勾配を伝播させないようにしようということ．  
なぜなら，$\Delta_\theta r_{v-adv}$がパラメータ$\theta$に対して非常に不安定であるため，固定しないと有効な正則化項を得られないから．  
tensorflow では，`tf.stop_gradient`で実装可能

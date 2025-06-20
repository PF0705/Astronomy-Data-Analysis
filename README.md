# 천문학 데이터 시각화 및 분석 프로젝트

이 프로젝트는 공개된 천문학 데이터를 이용해 H-R 다이어그램을 시각화하고, 구상성단과 산개성단의 H-R diagram 차이를 비교하며, 혜성의 궤도 데이터를 분석하는 과정입니다. 모든 분석은 Python과 Jupyter Notebook을 사용하여 수행되었습니다. Python을 이용하여 여러 데이터를 불러오고, 읽고 시각화하여 분석하는 과정(numpy, matplotlib, pandas 활용)을 보다 심도 있게 학습하는 것을 중점으로 두었으며, 전공 지식을 활용하여 유의미한 결과를 확인하려고 노력했습니다.

---

## 이론적 배경

### 1. HR Diagram

HR 다이어그램(Hertzsprung–Russell diagram)은 별의 절대등급 또는 광도와 온도(또는 색지수) 사이의 관계를 나타내는 산점도입니다. 별의 진화 단계(main sequence, red giant, white dwarf 등)를 시각적으로 이해할 수 있습니다.

### 2. 산개성단과 구상성단

- **산개성단(Open Cluster)**: 은하 평면에 분포하는 나이가 어린 성단으로, 별들이 흩어져 있는 모양입니다.
- **구상성단(Globular Cluster)**: 은하 헤일로에 분포하는 나이가 많은 성단으로, 구형 구조를 가집니다. 

### 3. 혜성 데이터 분석

혜성의 궤도 요소(e.g. eccentricity, inclination 등)를 통해 태양계를 공전하는 혜성들의 특징을 분석할 수 있습니다. KMeans 클러스터링을 통해 혜성군 분류도 시도했습니다.

---

## 데이터 설명

| 파일명 | 설명 |
|--------|------|
| `hipparcos-voidmain.csv` | Hipparcos 데이터 기반 항성 목록 |
| `ngc188_data.csv` | 산개성단 NGC 188의 별 데이터 |
| `tuc47_data.csv` | 구상성단 47 Tuc의 별 데이터 |
| `CometEls.txt` | 여러 혜성의 데이터 |

---

## 사용한 기술 및 라이브러리

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans
```
---

## 프로젝트

### 1. H-R Diagrams of Stars and Clusters

#### 1 ) HRD of Stars

- Theory

성단의 구분 없이 가능한 많은 별들에 대해 H-R도를 그리고 보편적 H-R도의 형태와 비교, 이론적인 별들의 종족과 진화 단계 등을 파악하고자 하였습니다.

사용한 데이터는 Hipparcos 위성에 의해 작성된 카탈로그 데이터입니다. 데이터의 구성(pandas로 읽어와 출력한 column 이름)은 다음과 같습니다.

`'Catalog', 'HIP', 'Proxy', 'RAhms', 'DEdms', 'Vmag', 'VarFlag', 'r_Vmag', 'RAdeg', 'DEdeg', 'AstroRef', 'Plx', 'pmRA', 'pmDE', 'e_RAdeg', 'e_DEdeg', 'e_Plx', 'e_pmRA', 'e_pmDE', 'DE:RA', 'Plx:RA', 'Plx:DE', 'pmRA:RA', 'pmRA:DE', 'pmRA:Plx', 'pmDE:RA', 'pmDE:DE', 'pmDE:Plx', 'pmDE:pmRA', 'F1', 'F2', '---', 'BTmag', 'e_BTmag', 'VTmag', 'e_VTmag', 'm_BTmag', 'B-V', 'e_B-V', 'r_B-V', 'V-I', 'e_V-I', 'r_V-I', 'CombMag', 'Hpmag', 'e_Hpmag', 'Hpscat', 'o_Hpmag', 'm_Hpmag', 'Hpmax', 'HPmin', 'Period', 'HvarType', 'moreVar', 'morePhoto', 'CCDM', 'n_CCDM', 'Nsys', 'Ncomp', 'MultFlag', 'Source', 'Qual', 'm_HIP', 'theta', 'rho', 'e_rho', 'dHp', 'e_dHp', 'Survey', 'Chart', 'Notes', 'HD', 'BD', 'CoD', 'CPD', '(V-I)red', 'SpType', 'r_SpType'`

여기서 사용한 주요 데이터를 간략하게 살펴보자면,

`'Plx'` : Parallax(연주시차). 보다 정확한 데이터 사용을 위해 관측된 시차가 없는 별들을 필터링하고 절대등급을 계산할 때 사용하였습니다.

`'B-V'` : 색지수. Blue filter로 관측한 등급에서 Visual(또는 Yellow) filter로 관측한 등급을 뺀 값으로, 온도가 클수록 작은 값을 나타냅니다. 온도를 계산하지 않고도 경향성을 나타낼 수 있어 편리하며 보편적인 H-R도 plot에서 빈번하게 사용됩니다. 코드에서는 log T(log temperature) 계산과 CMD(Color-Magnitude Diagram, H-R도의 온도 축이 색지수인 버전)의 축에 사용하였습니다.

`'Vmag'` : V filter에서 관측한 겉보기 등급을 뜻하며, 절대등급 계산에 사용하였습니다.

이 데이터로부터 계산할 수 있는 물리량은 다음과 같습니다.

`'AbsMag'` : 절대등급, 즉 별이 10pc 거리에 있을 때의 등급을 뜻하며 $M = m - 5 \left( \log_{10}\left( \frac{1000}{\pi} \right) - 1 \right)$로 계산할 수 있는데, 이때 $m$은 겉보기 등급(데이터의 Vmag), $\pi$는 시차(데이터의 Plx)를 나타냅니다.

`'log_L'` : 태양과 비교한 해당 별의 광도의 비율을 뜻하며 $\log_{10} \left( \frac{L}{L_\odot} \right) = -0.4 \left( M - 4.83 \right)$로 계산할 수 있습니다. $M$은 별의 절대등급, $L$은 별의 광도, $L_\odot$은 태양의 광도를 나타내며, 4.83은 태양의 절대등급 값입니다.

`'log_T'` : 표면온도에 로그를 취한 값입니다. 표면온도는 $T_{\text{eff}} = 4600 \left( \frac{1}{0.92(B - V) + 1.7} + \frac{1}{0.92(B - V) + 0.62} \right)$(Ballesteros 공식)로 계산할 수 있으며 $B-V$는 색지수를 뜻합니다.

- Result

계산한 데이터들로부터 그린 HRD, CMD는 다음와 같습니다.

![H-R Diagram](results/H_R_Diagram.png)

*[fig. 1] HRD from Hipparcos data*

![C-M Diagram](results/C_M_Diagram.png)

*[fig. 2] CMD from Hipparcos data*

가장 보편적인 CMD의 모습은 다음과 같습니다.

![general C-M Diagram](results/C_M_D_theory.png)

*[fig. 3] General CMD*

- Discussion

HRD와 CMD를 비교해보면 모양이 비슷하나 축의 스케일이 다르다는 것을 알 수 있습니다. 이는 위의 수식에도 잘 맞는 결과입니다.

직접 그린 CMD와 보편적 CMD는 거의 비슷한 모양을 나타냅니다. 다만 CMD를 그릴 때 proper motion 등을 이용해 필터링하지 않았기 때문에 그런지 조금 더 위아래로 퍼진 모습을 보입니다. 전체적으로 별의 주계열과 거성가지 등이 잘 나타나 있어 이론적인 별의 진화 경로와도 매칭이 잘 됩니다. 백색왜성과 밝은 주계열성 부분은 상대적으로 적게 나타나는 것을 보아, 계산 오차 등을 감안하더라도 사용한 데이터가 한정적이기 때문에 아주 젊거나 아주 늙은 성단의 데이터는 포함하지 못했음을 추측할 수 있습니다.

#### 2 ) HRD of Clusters

- Theory

구상성단과 산개성단은 발견되는 위치, 나이, 색 등에 차이를 보입니다. 구상성단은 나이가 많기 때문에 주계열이 짧고 어두우며 붉은 계열의 별이 많습니다. 산개성단은 비교적 주계열에 있는 별이 많고 푸른 계열의 별이 나타납니다. 이를 확인해보기 위해 산개성단인 NGC188, 구상성단인 47 TUC의 별 데이터를 이용해 각각 CMD를 그려보고, 겹쳐서 비교해보았습니다. 기본적인 물리량은 위에서 사용한 것과 비슷하나 이번 데이터는 절대등급으로 변환이 어려워 y축으로 g band의 겉보기 등급을 사용하였습니다.

- Result

두 성단 각각의 CMD는 다음과 같습니다.

![C-M Diagram each](results/C_M_D_clusters.png)

*[fig. 4] CMDs of each clusters*

두 CMD를 겹쳐 그린 결과는 다음과 같습니다.

![C-M Diagram comparison](results/C_M_D_comparison.png)

*[fig. 5] CMD for comparison*

- Discussion

결과를 보면 위에서 본 CMD의 형태와 다른 것을 볼 수 있습니다. 이는 스케일의 차이가 존재하는 점, cluster member 필터링을 하지 않은 데이터라는 점과 앞에서 말한 겉보기 등급 사용으로 이론과 차이가 있다는 점을 이유로 들 수 있습니다. 그래도 주계열과 그 turnoff(주계열이 끝나는 시점) point는 구별이 가능하고, 겹친 결과를 봤을 때 전체적으로 구상성단이 오른쪽(붉고 온도가 낮은 영역)으로 치우친 개형을 보이는 것으로 이론과 맞는 데이터라고 판단할 수 있을 것 같습니다. 적절한 필터링 등이 더해진다면 더 깔끔한 데이터를 얻을 수 있을 것이고, 전환점 등이 더 잘 보이도록 스케일 조정을 하는 것도 개선할 수 있는 사항 중 하나입니다.

### 2. Comet Data Analysis

사용한 데이터의 구성은 다음과 같습니다.

`'Comet_Number', 'Orbit_Type', 'Provisional_Designation', 'Perihelion_Year', 'Perihelion_Month', 'Perihelion_Day', 'Perihelion_Distance_AU', 'Eccentricity', 'Arg_of_Perihelion_deg', 'Asc_Node_deg', 'Inclination_deg', 'Epoch_Year', 'Epoch_Month', 'Epoch_Day', 'Absolute_Magnitude', 'Slope_Parameter', 'Designation_and_Name', 'Reference'`

사용한 주요 데이터를 간략하게 살펴보자면,

`'Perihelion_Year'` : 혜성이 근일점을 통과한(혹은 통과할거라고 예측되는) 연도 데이터입니다.

`'Perihelion_Distance_AU'` : 혜성의 근일점 거리입니다. 주기와의 비례 관계를 가지므로 주기를 간접적으로 계산할 수 있으며, 혜성의 활동성과도 연관이 어느 정도 있는 물리량입니다.

`'Eccentricity'` : 이심률, 즉 궤도가 찌그러져있는 정도를 뜻하는 물리량이며 근일점 거리 등의 여러 물리량과 관련이 있습니다. 0~1 사이 범위를 가지고 1로 갈수록 더 찌그러진 궤도입니다. 1보다 큰 경우 타원이 아닌 쌍곡선 등의 궤도를 뜻합니다.

`'Inclination_deg'` : 궤도 경사각을 뜻합니다.

#### 1 ) Time series Analysis(Perihelion Point Passing)

- Theory

혜성이 근일점을 지나는 것이 관측되거나(과거), 예측되는(미래) 개수를 연도별로 나타내어 관측 기술의 발전이나 혜성 주기 등의 경향성을 파악할 수 있습니다.

- Result

히스토그램으로 나타내면 다음과 같은 결과를 볼 수 있습니다.

![time analysis](results/passage_year.png)

*[fig. 6] Perihelion passage year histogram*

- Discussion

결과 사진을 보면 현재로 올수록 관측된 혜성의 개수가 증가하는 것을 확인할 수 있는데, 이는 관측 기술의 발전을 시사합니다. 하지만 2006년에 이례적으로 많은 수의 혜성이 관측된 것은 예측에서 벗어난 결과입니다. 관측 기술 덕분이었다면 그 이후에 그보다 많은 혜성이 관측되었어야 하는데, 그렇지는 않은 것을 보아 많은 주기 혜성들의 주기가 우연히 맞아떨어진 것 등의 원인이 있었을 것이라고 생각할 수 있습니다.

#### 2 ) Relation between Physical Quantities

- Theory

데이터에 있는 물리량 사이의 상관관계가 있는지 의문이 들어 산점도 plot과 fitting line을 나타내고 Pearson 상관계수(r)를 계산해보았습니다. 이 상수의 값에 따라 상관관계의 경향성이 대략적으로 결정됩니다.

| r 값 범위              | 해석                         |
|------------------------|------------------------------|
| r = +1.0               | 완전한 양의 선형 상관관계     |
| +0.7 ≤ r < +1.0        | 강한 양의 상관관계            |
| +0.4 ≤ r < +0.7        | 중간 정도의 양의 상관관계     |
| +0.1 ≤ r < +0.4        | 약한 양의 상관관계            |
| -0.1 < r < +0.1        | 거의 없음 (무상관)            |
| -0.4 < r ≤ -0.1        | 약한 음의 상관관계            |
| -0.7 < r ≤ -0.4        | 중간 정도의 음의 상관관계     |
| -1.0 ≤ r ≤ -0.7        | 강한 음의 상관관계            |
| r = -1.0               | 완전한 음의 선형 상관관계     |

가장 관련이 있을 것 같은 근일점 거리와 절대밝기, 이심률과 공전주기 사이의 상관관계를 분석해보았습니다.


- Result

근일점 거리와 절대밝기 사이의 산점도와 선은 다음과 같이 나타납니다.

![distance, magnitude comparison](results/d_vs_Mag.png)

*[fig. 7] Perihelion Distance vs Absolute Magnitude plot*

Pearson 상관계수 출력 결과는 다음과 같습니다.

`Pearson correlation constant r = -0.415`

이심률과 공전주기 사이의 산점도와 선은 다음과 같이 나타납니다.

![ecc, period comparison](results/ecc_vs_P.png)

*[fig. 8] Eccentricity vs Orbital Period plot*

Pearson 상관계수 출력 결과는 다음과 같습니다.

`Pearson correlation constant r = 0.987`

- Discussion

근일점 거리와 절대밝기 사이의 경우, 상관계수의 값이 -0.7과 -0.4의 중간 값을 가지기 때문에 두 물리량은 약하게 반비례한다는 결론을 내릴 수 있고, 실제로 선이 우하향하는 것으로 확인할 수 있습니다. 절대밝기가 밝을수록 혜성의 활동성이 더 크다고 말하는데, 태양에 더 가까이 지날수록 그럴 가능성이 높다고 하는 연구도 존재합니다. 다만 데이터 점이 퍼져있는 것을 보아 이상치의 영향을 받았을 가능성을 무시할 수 없고, 이론적으로도 확인된 비례관계가 없으니 약한 경향성(예외 다수 존재)으로만 참고하면 좋을 것 같습니다. 

이심률과 공전주기 사이의 경우, 상관계수의 값이 1에 가까운 값으로 강한 비례관계를 보입니다. fitting한 선을 보면 우상향하고, 선에서 벗어나있는 점들도 이심률이 커질수록 주기가 커지는 경향성을 보입니다. 이는 이론적으로도 이심률이 클수록 같은 장반경에 대해 더 긴 둘레를 가지고(속도 같을 경우 시간 더 소요), 이심률이 매우 클 경우 쌍곡선 궤도(돌아오지 않음, 이론적인 주기 $\infty$) 등이 될 수 있기 때문에 잘 맞는 결과라고 할 수 있습니다. 

#### 3 ) Eccentricity Distribution

- Theory

관측된 혜성들의 이심률 분포를 히스토그램으로 나타내보았습니다. 이심률이 1보다 큰 혜성들은 지나가면 다시 돌아오지 않기 때문에 관측에는 우연이 필요하고, 따라서 많이 없을 것이라고 예측하고 분석을 진행하였습니다.

- Result

이심률 분포 히스토그램은 다음과 같이 출력되었습니다.

![eccentricity distriution](results/ecc_distribution.png)

*[fig. 9] Eccentricity Distribution Histogram*

- Discussion

예측대로 이심률이 1 미만인 혜성이 더 많이 나타났으나, 특이한 점은 정확히 이심률이 1인 혜성이 많다는 것이었습니다. 조사해보니 이 원인으로는 크게 2가지가 있었습니다. 첫 번째는 장주기 혜성들의 근원이라고 추측하는 오르트구름(Oort cloud)에서 온 혜성들은 이심률이 매우 큰 긴 타원 궤도 또는 쌍곡선 궤도에 가까운 경우가 대다수기 때문에 이심률이 거의 1에 가깝기 때문입니다. 오르트구름은 매우 멀기 때문에 태양까지 도달하기 위해서는 큰 이심률 궤도가 필요하고, 이를 관측하면 이심률이 1에 가까워 근사하면 1로 입력되는 경우가 많다고 합니다. 그리고 실제로 장주기 혜성이 굉장히 많기 때문에 이러한 분포가 나오는 것이라고 추측할 수 있습니다. 두 번째로는 관측의 한계입니다. 혜성의 궤도 전체를 관측하기란 사실상 불가능하기 때문에 태양에 근접한 순간의 데이터를 가지고 궤도를 fitting하는 작업이 필요한데, 이 과정에서(특히 관측 시간이 짧을 때) 이심률이 1 또는 그 이상으로 계산되는 경우가 잦고, 1로 가정한 후 정밀화하는 작업을 실행하기도 하기 때문에 정확히 1로 입력되어있는 혜성이 생각보다 많고, 이 혜성들은 사실 1에 가까운 이심률을 가지고 있는 것이라는 추측이 가능합니다.

#### 4 ) K-means Clustering

- Theory

혜성을 물리량을 이용해 분류할 수 있는지(구분지어지는지) 알아보기 위해 K-means clustering을 이용해 분석해보았습니다. 이심률과 근일점 거리, 궤도 경사각과 근일점 거리 사이에서 혜성들이 구분 가능한 군집을 이루는지 판단하는 작업입니다. 사용한 군집의 수는 3개입니다. 군집이 구분되게 형성될수록 혜성이 그 물리량 하에서 독립적인 상태를 가진다고 판단할 수 있습니다.

- Result

이심률과 근일점 거리를 이용한 K-means clustering 결과는 다음과 같습니다.

![cluster1](results/clustering.png)

*[fig. 10] K-means Clustering(Eccentricity/Perihelion distance)*

궤도 경사각과 근일점 거리를 이용한 K-means clustering 결과는 다음과 같습니다.

![cluster2](results/clustering2.png)

*[fig. 11] K-means Clustering(Orbit inclination/Perihelion distance)*

- Discussion

이심률과 근일점 거리 사이의 군집들을 보면 초록색과 노란색은 서로 명확히 구분되지만 초록색과 보라색은 그렇지 않습니다. 보라색은 이심률이 1 근처인 혜성들이기 때문에 장주기 혜성에 대해서 어떤 결과를 내기는 어려울 것 같고, 그렇지 않은 단주기 혜성들 사이에는 무언가 연관성이 있을 것이라고 생각할 수 있습니다.

궤도 경사각과 근일점 거리 사이의 군집들을 보면 서로 구분되지 않거나, 너무 퍼져있어 군집으로 볼 수 없는 결과들이 나온 것을 보아 유의미한 구분은 어려울 것이라고 예측할 수 있습니다.

클러스터링 후 추가적인 통계적 분석은 관련 지식의 부족으로 하지 못했는데, 그런 분석을 하거나 군집의 수를 다르게 하여 비교해 본다면 더 명확한 결과를 낼 수 있을 것 같습니다.

---

## 참고문헌, 코드, 사용 데이터

- Reference

Janes, Kenneth A., and Sadia Hoq. 2011. “A Quantitative Analysis of Distant Open Clusters.” *The Astronomical Journal* 141 (3): 92. (https://iopscience.iop.org/article/10.1088/0004-6256/141/3/92)

Mazzotta Epifani, E., P. Palumbo, and L. Colangeli. 2009. “A Survey on the Distant Activity of Short Period Comets.” *Astronomy & Astrophysics* 508 (2): 1031–1044. (https://www.aanda.org/articles/aa/abs/2009/47/aa12611-09/aa12611-09.html)

https://www.wired.com/story/so-youve-found-a-comet-with-a-weird-orbit/

Chat GPT(Various computations, Pearson constant, K-means clustering, etc.)

- Code

코드는 anaconda 가상환경을 사용해 jupyter notebook 파일로 작성되었으며, repository의 notebooks 폴더에서 보실 수 있습니다. 사용한 라이브러리는 위에 명시되어 있습니다.

- Data

데이터는 첨부를 시도하였으나 github의 용량 한계로, 출처 링크를 대신 남깁니다. 필요할 경우 아래 링크에서 다운받아주시면 됩니다.

https://www.kaggle.com/datasets/konivat/hipparcos-star-catalog?resource=download (hipparcos-voidmain.csv / 전체 HRD, CMD에 사용한 데이터)

https://www.kaggle.com/datasets/esatakkasoglu/star-clusters-47-tuc-globular-and-ngc188-open/data (ngc188_data.csv, tuc47_data.csv / 성단 분석에 사용한 데이터)

https://www.minorplanetcenter.net/iau/MPCORB/CometEls.txt (CometEls.txt / 혜성 분석에 사용한 데이터)

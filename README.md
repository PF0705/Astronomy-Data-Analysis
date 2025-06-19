# Astronomy-Data-Analysis
수학과프로그래밍 기말 프로젝트 - 2023135003 서유리

# 🪐 천문학 데이터 시각화 및 분석 프로젝트

이 프로젝트는 공개된 천문학 데이터를 이용해 HR 다이어그램을 시각화하고, 구상성단과 산개성단의 분포 차이를 비교하며, 혜성의 궤도 데이터를 분석합니다. 모든 분석은 Python과 Jupyter Notebook을 사용하여 수행되었습니다.

---

## 🧠 이론적 배경

### 1. HR Diagram
HR 다이어그램(Hertzsprung–Russell diagram)은 별의 절대등급 또는 광도와 온도(또는 색지수) 사이의 관계를 나타내는 산점도입니다. 별의 진화 단계(main sequence, red giant, white dwarf 등)를 시각적으로 이해할 수 있습니다.

### 2. 산개성단과 구상성단
- **산개성단(Open Cluster)**: 젊고 은하면에 분포, 성단의 별들이 흩어져 있음.
- **구상성단(Globular Cluster)**: 오래되고 구형 구조, 은하 헤일로에 분포.

### 3. 혜성 궤도 분석
혜성의 궤도 요소(e.g. eccentricity, inclination 등)를 통해 태양계를 공전하는 특징을 분석할 수 있습니다. KMeans 클러스터링을 통해 혜성군 분류도 시도했습니다.

---

## 📁 데이터 설명

| 파일명 | 설명 |
|--------|------|
| `hipparcos-voidmain.csv` | Hipparcos 데이터 기반 항성 목록 |
| `ngc188_data.csv` | 산개성단 NGC 188의 별 데이터 |
| `tuc47_data.csv` | 구상성단 47 Tuc의 별 데이터 |
| `CometEls.txt` | 여러 혜성의 궤도 요소 데이터 |

---

## 🧪 사용한 기술 및 라이브러리

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans

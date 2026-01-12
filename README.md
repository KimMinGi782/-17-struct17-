# -17-struct17-
신호가 아닌 구조로 생존과 축적을 설계하는 시스템.(Structure-first systems for trading, risk control, and accumulation. Focused on architecture, not signals.)
# NUMERAI A+++ All-in-One Unified Pipeline (2026-01)

Numerai Tournament용 **완전 통합형 예측 파이프라인**입니다.  
단일 코드에서 다음을 모두 수행합니다.

- 다중 Exposure Pack 생성 (베타/클러스터/PCA/키워드/StableCorr)
- Target별 최근 era 기반 neutralization proportion CV
- Kurtosis-aware post-processing (fat-tail 제어)
- 최근 era 기반 dynamic target weighting
- Per-target 학습/예측/후처리/블렌드
- 최종 `submission.csv` 생성

> ⚠️ 이 저장소는 **연구/구조 공개 목적**입니다.  
> 실전 운영 최적화(파라미터 고정 세트, 비공개 튜닝 루틴 등)는 포함하지 않습니다.

---

## 핵심 구성

### 1) Exposure Pack (자동 생성)
다음 5종 exposure를 결합합니다.

- **Beta Proxy Features**: 최근 era 기준 target 상관 상위 feature
- **Keyword Pack**: sector/industry/macro/style 등 키워드 기반 feature
- **Correlation Clustering Reps**: 고상관 그룹에서 대표 feature만 선택
- **Stable Correlation Pack**: 최근 era에서 평균↑/분산↓ 상관 feature
- **PCA Exposures**: feature 공간을 저차원으로 요약한 PC 점수

→ 생성된 exposure는 **era-wise neutralization(노출 제거)**에 사용됩니다.

---

### 2) Target-wise Neutralization Proportion (CV)
- target별로 neutralization proportion을 **era 기반 CV**로 선택합니다.
- 최근 era를 우선하여 **안정성(variance 감소)**을 목표로 합니다.

---

### 3) Kurtosis-Aware Postprocess
- era-wise rank → inverse normal(근사) → tanh squash
- excess kurtosis에 따라 **dynamic clipping** 적용

→ 분포를 안정화하고 fat-tail 리스크를 완화합니다.

---

### 4) Dynamic Target Weighting
- validation의 최근 era corr을 기반으로 target별 가중치를 자동 생성합니다.
- `power`로 가중 집중도를 조절합니다.

---

## 요구 사항

- Python 3.9+
- numpy
- pandas
- scipy ❌ (사용하지 않음)

---

## 데이터 전제

DataFrame에 아래 컬럼이 있어야 합니다.

- `id`
- `era`
- `feature_*`
- `target_*` (train/val에만 존재, live에는 없음)

또한, `train_df`, `val_df`, `live_df`는 **사용자가 직접 로드**해야 합니다.

---

## 빠른 시작

```python
targets = [
    "target_teager2b_20",
    "target_teager2_20",
    "target_nomi_v4_20",
]

submission_df = run_numerai_all_in_one(
    train_df=train_df,
    val_df=val_df,
    live_df=live_df,
    targets=targets,
)

submission_df.to_csv("submission.csv", index=False)

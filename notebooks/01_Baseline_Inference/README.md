# 🔍 Step 1. Baseline Inference: Vehicle Detection

본격적인 파손 탐지(Damage Detection) 모델을 개발하기 전, **COCO 데이터셋으로 사전 학습된(Pre-trained) YOLOv8x 모델**이 우리가 구축한 데이터셋에 대해 **'차량(Car)' 객체를 얼마나 안정적으로 검출하는지** 베이스라인 성능을 점검

## 🎯 Objective (실험 목표)
1.  **Detectability Check:** 심하게 파손된 차량(Damaged Car)도 일반적인 'Car' 클래스로 인식할 수 있는지 확인.
2.  **False Positive Check:** 배경(Background) 이미지에서 엉뚱한 물체를 차량으로 오인하지 않는지 확인.
3.  **Threshold Tuning:** 최적의 Confidence Threshold 값을 탐색하여 Recall(재현율)과 Precision(정밀도)의 균형점 확인.

## 🛠 Experiment Setup
* **Model:** YOLOv8x (Pre-trained on COCO)
* **Input Resolution:** Original Size (No Resizing)
* **Task:** Object Detection (Class: Car)
* **Environment:** Google Colab Pro (T4/A100 GPU)

## 📊 1. Visual Inspection 

| class | count | ratio | description |
| :---: | :---: | :---: | :---: |
| damaged | 1200 | 0.61318 | vehicle |
| normal | 157 | 0.08022 | vehicle |
| background | 600 | 0.30659 | non-vehicle | 
| total | 1957 | 1.0 | all-image | 

* 차량 : 비차량 = 1357 : 600 =  0.694 : 0.306

### ✅ Case 1: Normal & Damaged Car
파손 여부와 관계없이 모델이 차량의 BBox를 정확히 잡는지 확인

| Normal Car (정상) | Damaged Car (파손) |
| :---: | :---: |
| ![Normal](./results/01_detection/sample_normal.png) | ![Damaged](./results/01_detection/sample_damaged.png) |
> **Result:** YOLOv8x 모델은 찌그러지거나 긁힌 차량도 시각적 특징(바퀴, 형태 등)을 통해 **'Car'로 정확히 인식**함을 확인

### ✅ Case 2: Background (Negative Sample)
차량이 없는 이미지에서의 오탐지 여부

| Background (배경) |
| :---: |
| ![Background](./results/01_detection/sample_bg.png) |
> **Result:** (여기에 결과 작성, 예: 벤치를 차량으로 오인하는 경우가 있었으나 Confidence 0.5 이상에서 제거됨 등)

---

## 📈 2. Quantitative Analysis (정량적 평가)
Confidence Threshold(임계값)를 변경해가며 전체 데이터셋에 대한 검출 성능을 테스트

| Conf Threshold | Accuracy | average inference speed | FPS |
| :---: | :---: | :---: | :---: |
| **0.25** |85.49%| 47.94 ms/장 | 20.86 FPS |
| **0.10** |88.71%| 48.23 ms/장 | 20.73 FPS |

| **0.25** | **0.10** |
| :---: | :---: |
| ![025](./results/01_detection/confusion_matrix_025.png) | ![010](./results/01_detection/confusion_matrix_010.png) |


| Conf Threshold | Class | Precision | Recall | f1 | 비고 |
| :---: | :---: | :---: | :---: | :--- | :--- |
| **0.25** |Non-Vehicle| 0.68 | 0.99 | 0.81 | precision 낮음 |
| **0.25** |Vehicle| 0.99 | 0.80 | 0.88 | recall 낮음 |
| **0.10** |Non-Vehicle| 0.74 | 0.96 | 0.84 | precision 낮음 |
| **0.25** |Vehicle| 0.91 | 0.89 | 0.89 | recall 낮음 |

### 💡 Findings
* **Confidence 0.25:** 재현율(Recall)은 높으나, 배경 이미지에서 일부 오탐지(False Positive) 발생.
* **Confidence 0.50:** 파손된 차량도 놓치지 않으면서 오탐지가 현저히 줄어듦. 베이스라인으로 적합 판단.

---

## 📝 Conclusion & Next Step
* **결론:** Pre-trained YOLOv8x 모델은 별도의 Fine-tuning 없이도 '차량 인식(Vehicle Detection)' 단계에서 충분한 성능을 보여줍니다. 파손된 차량도 'Car'로 잘 인식하므로, 1단계(차량 찾기) 모델로 채택 가능합니다.
* **Next Step:** 이제 차량 영역(ROI) 안에서 **'어디가 파손되었는지(Damage Localization)'**를 찾는 모델을 학습시키는 **Step 2. Damage Detection Fine-tuning**을 진행합니다.

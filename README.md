# BEVFusion on CARLA

> **BEVFusion**을 nuScenes로 학습하고, CARLA 시뮬레이터에서 3D 객체 인지를 검증합니다.

```mermaid
flowchart LR
    subgraph TRAIN["🏋️ Training  —  nuScenes"]
        direction TB
        NS_L["📡 LiDAR"]
        NS_C["📷 Camera"]
    end
    BEV["🔀 BEVFusion<br/>BEV Fusion · 3D Det. Head"]
    subgraph INFER["🚗 Inference  —  CARLA"]
        direction TB
        CA_L["📡 LiDAR"]
        CA_C["📷 Camera"]
    end
    OUT["📦 3D Objects<br/>bbox · class · velocity"]
    NS_L & NS_C --> BEV
    BEV -. "pretrained weights" .-> OUT
    CA_L & CA_C --> OUT
    classDef data   fill:#0a2233,stroke:#00d4ff,color:#00d4ffee,font-weight:bold
    classDef model  fill:#0a2018,stroke:#34d399,color:#34d399ee,font-weight:bold
    classDef output fill:#0a1f0a,stroke:#4ade80,color:#4ade80ee,font-weight:bold
    class NS_L,NS_C,CA_L,CA_C data
    class BEV model
    class OUT output
```

---

## 📌 프로젝트 소개

본 프로젝트는 멀티모달 3D 객체 인지 모델인 **BEVFusion**을 활용하여, 자율주행 시뮬레이터 **CARLA** 환경에서의 인지 성능을 검증하는 것을 목적으로 합니다.

BEVFusion은 LiDAR와 Camera 두 센서를 **Bird's Eye View(BEV) 공간에서 융합**함으로써, 단일 센서 대비 강인한 3D 객체 탐지 성능을 달성합니다. 실제 도로 데이터셋(nuScenes)으로 학습된 모델을 시뮬레이션 환경(CARLA)에 적용함으로써 **sim-to-real 전이 가능성**을 탐색합니다.

| 항목 | 내용 |
|------|------|
| 모델 | BEVFusion (MIT) |
| 학습 데이터 | nuScenes |
| 추론 환경 | CARLA Simulator v0.9.x |
| 입력 센서 | LiDAR (32-beam) + Camera (6×RGB) |
| 출력 | 3D Bounding Box · Class · Velocity |

---

## 🏋️ Training

nuScenes 데이터셋을 기반으로 BEVFusion 모델을 학습합니다.

```bash
# 학습 실행
python tools/train.py configs/bevfusion/bevfusion-3d-camera-lidar.py
```

| 설정 | 값 |
|------|----|
| Dataset | nuScenes v1.0 full data |
| Epochs | 20 |
| Batch Size | 2 |
| Optimizer | AdamW |
| LR Scheduler | CosineAnnealing |
| GPU | RTX 5090 |

---

## 🚗 Inference on CARLA

학습된 가중치를 바탕으로 CARLA 시뮬레이터에서 실시간 추론을 수행합니다.

```bash
# CARLA 서버 실행
./CarlaUE4.sh -world-port=2000

# 추론 실행
python tools/carla_inference.py \
    --config configs/bevfusion/bevfusion-3d-camera-lidar.py \
    --checkpoint checkpoints/bevfusion_nuscenes.pth \
    --carla-host localhost \
    --carla-port 2000
```

CARLA에서 수집한 LiDAR 포인트클라우드와 멀티뷰 카메라 이미지를 nuScenes 포맷으로 변환한 뒤 모델에 입력합니다.

---

## 📊 Results

### BEVFusion on CARLA

<img width="1450" height="474" alt="bevfusion result" src="https://github.com/user-attachments/assets/4c19970d-ae73-4f4d-bb01-eac57e2dcef9" />

> **camera+lidar bev fusion object detection perception**

### CARLA Inference

<img width="960" height="618" alt="carla inference" src="https://github.com/user-attachments/assets/b2050dd8-8abf-4984-8191-ef9aa5aeea80" />

> **carla object detection inference**

### Detection Performance

| Metric | nuScenes (val) | CARLA |
|--------|---------------|-------|
| mAP | 68.5 | - |
| NDS | 71.4 | - |
| Car AP | 84.3 | - |
| Pedestrian AP | 82.1 | - |
| Cyclist AP | 58.2 | - |

> CARLA 성능 수치는 추론 실험 완료 후 업데이트 예정입니다.

---

## ⚡ Lightweight BEVFusion on Jetson (On-Board)

> 카메라 백본을 **Swin-T → ResNet-50** 으로 교체하여 모델을 경량화하고, **NVIDIA Jetson** AGX 보드 환경에서 실시간 추론을 검증합니다.

```mermaid
flowchart LR
    subgraph TRAIN["🏋️ Training  —  nuScenes"]
        direction TB
        NS_L["📡 LiDAR"]
        NS_C["📷 Camera"]
    end

    subgraph MODEL["⚡ BEVFusion-Lite"]
        direction TB
        CHANGE["Swin-T → ResNet-50<br/>backbone 경량화"]
        TRT["TensorRT<br/>INT8 Quantization"]
        CHANGE --> TRT
    end

    subgraph JETSON["🖥️ Jetson — AGX-Board"]
        direction TB
        JL["📡 LiDAR"]
        JC["📷 Camera"]
    end

    OUT["📦 3D Objects<br/>bbox · class · velocity"]

    NS_L & NS_C --> MODEL
    MODEL -. "pretrained weights" .-> OUT
    JL & JC --> OUT

    classDef data    fill:#0a2233,stroke:#00d4ff,color:#00d4ffee,font-weight:bold
    classDef model   fill:#1a0f00,stroke:#f97316,color:#f97316ee,font-weight:bold
    classDef jetson  fill:#1a0a2a,stroke:#a78bfa,color:#a78bfaee,font-weight:bold
    classDef output  fill:#0a1f0a,stroke:#4ade80,color:#4ade80ee,font-weight:bold

    class NS_L,NS_C data
    class CHANGE,TRT model
    class JL,JC jetson
    class OUT output
```

| 항목 | 기존 BEVFusion | BEVFusion-Lite |
|------|--------------|----------------|
| Camera Backbone | Swin-T | ResNet-50 |
| 파라미터 수 | ~70M | ~35M |
| 추론 환경 | RTX 5090 | Jetson Orin |
| 최적화 | - | TensorRT INT8 |
| 목표 FPS | - | ≥ 20 FPS |


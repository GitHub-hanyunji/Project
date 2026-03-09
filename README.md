
# BEVFusion on CARLA

> **BEVFusion**을 nuScenes로 학습하고, CARLA 시뮬레이터에서 3D 객체 인지를 검증합니다.

```mermaid
flowchart LR
    subgraph TRAIN["🏋️ Training  —  nuScenes"]
        direction TB
        NS_L["📡 LiDAR"]
        NS_C["📷 Camera"]
    end

    BEV["🔀 BEVFusion <br/> BEV Fusion · 3D Det. Head"]

    subgraph INFER["🚗 Inference  —  CARLA"]
        direction TB
        CA_L["📡 LiDAR"]
        CA_C["📷 Camera"]
    end

    OUT["📦 3D Objects <br/> bbox · class · velocity"]

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
### BEVFusion on CARLA
<img width="1450" height="474" alt="image" src="https://github.com/user-attachments/assets/4c19970d-ae73-4f4d-bb01-eac57e2dcef9" />

> **camera+lidar bev fusion object detection perception**

---
### Carla
<img width="960" height="618" alt="image" src="https://github.com/user-attachments/assets/b2050dd8-8abf-4984-8191-ef9aa5aeea80" />

> **carla object detection inference**


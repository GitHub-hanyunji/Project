# Project

```mermaid
flowchart LR
  flowchart LR
    %% ── CARLA Simulator ──
    subgraph CARLA["🌆 CARLA Simulator"]
        direction TB
        LIDAR["📡 LiDAR Sensor\n64-beam, 360°\n~100k pts/frame"]
        CAM["📷 Camera Array\n6× RGB, 1600×900\nsurround view"]
    end

    %% ── Preprocessing ──
    subgraph PRE["⚙️ Preprocessing"]
        direction TB
        VOX["🧊 Voxelization\n& Pillar Encoding\n0.2m voxel grid"]
        IMGPRE["🖼️ Image Preprocess\nResize + Normalize\n256×704 per cam"]
    end

    %% ── Backbone ──
    subgraph BACK["🔩 Backbone"]
        direction TB
        PTBACK["⚙️ PtCloud Backbone\nVoxelNet / PointPillars\nsparse 3D conv"]
        IMGBACK["⚙️ Image Backbone\nSwin-T / ResNet-50\nFPN neck"]
    end

    %% ── View Transform ──
    subgraph VT["🗺️ View Transform"]
        direction TB
        BEVL["🗺️ BEV Feature\nLiDAR branch\nH×W×C map"]
        BEVC["🗺️ BEV Feature\nCamera — LSS\nLift-Splat-Shoot"]
    end

    %% ── Fusion ──
    subgraph FUS["🔀 BEV Fusion"]
        FUSE["🔀 Concat + Conv\nunified BEV feature"]
    end

    %% ── Detection Head ──
    subgraph HEAD["🎯 Detection Head"]
        DH["🎯 CenterPoint Head\nheatmap-based\nanchor-free"]
    end

    %% ── Output ──
    subgraph OUT["📦 Output"]
        direction TB
        BBOX["📦 3D BBox\nx,y,z,l,w,h,θ"]
        CLS["🏷️ Class Label\ncar / ped / bike\n+ confidence"]
        VEL["💨 Velocity\nvx, vy — m/s\nfor tracking"]
    end

    %% ── Edges ──
    LIDAR --> VOX
    CAM   --> IMGPRE
    VOX   --> PTBACK
    IMGPRE --> IMGBACK
    PTBACK --> BEVL
    IMGBACK --> BEVC
    BEVL --> FUSE
    BEVC --> FUSE
    FUSE --> DH
    DH --> BBOX
    DH --> CLS
    DH --> VEL

    %% ── Styles ──
    classDef sensor   fill:#0a2233,stroke:#00d4ff,color:#00d4ff,font-weight:bold
    classDef pre      fill:#1a0a33,stroke:#a78bfa,color:#a78bfa,font-weight:bold
    classDef backbone fill:#2a0a1a,stroke:#f472b6,color:#f472b6,font-weight:bold
    classDef viewt    fill:#2a1a00,stroke:#fbbf24,color:#fbbf24,font-weight:bold
    classDef fusion   fill:#0a2018,stroke:#34d399,color:#34d399,font-weight:bold
    classDef head     fill:#0a1a22,stroke:#22d3ee,color:#22d3ee,font-weight:bold
    classDef output   fill:#0a1f0a,stroke:#4ade80,color:#4ade80,font-weight:bold

    class LIDAR,CAM sensor
    class VOX,IMGPRE pre
    class PTBACK,IMGBACK backbone
    class BEVL,BEVC viewt
    class FUSE fusion
    class DH head
    class BBOX,CLS,VEL output
  
```

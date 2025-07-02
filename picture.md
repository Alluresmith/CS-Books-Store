```mermaid
flowchart TD
    %% ---------- 输入 ----------
    subgraph DATA["✦ 数据管道"]
        Z[Z (原子序)] --> POS[坐标 pos]
        POS --> CELL[分子盒 cell]
        CELL --> MESH_POS[网格坐标 mesh_pos]
    end
    style DATA fill:#fafafa,stroke:#999,stroke-width:1px

    %% ---------- 短程图网络 ----------
    subgraph SHORT["① 短程 TensorNet 图网络"]
        AEMBED[TensorEmbed] --> INTL[Interaction × L] --> READOUT[Readout]
    end
    style SHORT fill:#e8f8ff,stroke:#3ba0e6,stroke-width:1px

    %% ---------- 原子→网格 ----------
    subgraph A2M["② A → M 投影 & 注入"]
        POOL[分子池化<br/>(mean)]
        EDGE[a2m radius_graph]
        PROJ1[a2m_proj]
        POOL --> PROJ1
        INTL -.-> POOL
        INTL -.-> EDGE --> PROJ1
    end
    style A2M fill:#fff7e6,stroke:#ffb44f,stroke-width:1px

    %% ---------- 网格长程 ----------
    subgraph MESH["③ 网格长程传播"]
        FNO[FNO3D<br/>/ Transformer]
    end
    style MESH fill:#f6e7ff,stroke:#c376ff,stroke-width:1px

    %% ---------- 网格→原子 ----------
    subgraph M2A["④ M → A 回写 & 残差"]
        PROJ2[m2a_proj] --> SCATTER[scatter-add] --> LN[LayerNorm+SiLU] --> RES[x_atom += ]
    end
    style M2A fill:#fff0f6,stroke:#ff7ab3,stroke-width:1px

    %% ---------- 输出 ----------
    subgraph OUT["⑤ 输出模块"]
        PRED[能量等 MLP] --> GRAD[autograd<br/>→ Force]
    end
    style OUT fill:#f4fffa,stroke:#70d67b,stroke-width:1px

    %% ---------- 主线连通 ----------
    READOUT --> A2M
    PROJ1 --> FNO --> PROJ2
    RES --> PRED

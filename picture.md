
flowchart TD
    %% ---------- 输入 ----------
    subgraph DATA["✦ 数据管道"]
        Z(Z 原子序) -->|concat| POS(坐标 pos)
        POS --> CELL(分子盒 cell)
        CELL --> MESH_POS(网格坐标 mesh_pos)
        style DATA fill:#fafafa,stroke:#aaa
    end

    %% ---------- 短程图网络 ----------
    subgraph SHORT["① 短程 TensorNet 图网络"]
        AEMBED(原子嵌入 TensorEmbed)
        INTL1(Interaction × L)
        READOUT(Readout)
        AEMBED --> INTL1 --> READOUT
        style SHORT fill:#e8f8ff,stroke:#3ba0e6
    end

    %% ---------- 原子→网格 ----------
    subgraph A2M["② A → M 投影 & 注入"]
        POOL(分子池化 mean)
        A2M_EDGE(radius\ngraph)
        PROJ1(a2m_proj (MLP/FC))
        POOL --> PROJ1
        INTL1 -.-> POOL
        INTL1 -.-> A2M_EDGE
        A2M_EDGE --> PROJ1
        style A2M fill:#fff7e6,stroke:#ffb44f
    end

    %% ---------- 网格长程 ----------
    subgraph MESH["③ 网格长程传播"]
        M_IN(Mesh init F<sub>0</sub>)
        M_FNO{FNO3D<br/>或<br/>Transformer}
        M_IN --> M_FNO
        style MESH fill:#f6e7ff,stroke:#c376ff
    end

    %% ---------- 网格→原子 ----------
    subgraph M2A["④ M → A 回写 & 残差"]
        PROJ2(m2a_proj)
        SCATTER(scatter-add)
        LN(LayerNorm + SiLU)
        RESIDUAL(x_atom +=)
        M_FNO --> PROJ2 --> SCATTER --> LN --> RESIDUAL
        style M2A fill:#fff0f6,stroke:#ff7ab3
    end

    %% ---------- 输出 ----------
    subgraph OUT["⑤ 输出模块"]
        PRED(能量/其他 MLP)
        GRAD("|autograd| → Force")
        RESIDUAL --> PRED --> GRAD
        style OUT fill:#f4fffa,stroke:#70d67b
    end

    %% ---------- 展开主线 ----------
    Z --> AEMBED

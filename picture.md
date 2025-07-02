```mermaid
flowchart TD
  %% ---------- 输入 ----------
  subgraph DATA["数据管道"]
    Z["Z (原子序)"] --> POS["坐标 pos"]
    POS --> CELL["分子盒 cell"]
    CELL --> MESH_POS["网格坐标 mesh_pos"]
  end

  %% ---------- 短程图网络 ----------
  subgraph SHORT["① 短程 TensorNet"]
    AEMBED["TensorEmbed"] --> INTL["Interaction × L"] --> READOUT["Readout"]
  end

  %% ---------- 原子→网格 ----------
  subgraph A2M["② A→M 投影"]
    INTL -.-> POOL["分子池化(mean)"]
    INTL -.-> EDGE["a2m radius_graph"]
    POOL --> PROJ1["a2m_proj"]
    EDGE --> PROJ1
  end

  %% ---------- 网格长程 ----------
  subgraph MESH["③ 网格长程传播"]
    PROJ1 --> FNO["FNO3D / Transformer"]
  end

  %% ---------- 网格→原子 ----------
  subgraph M2A["④ M→A 回写"]
    FNO --> PROJ2["m2a_proj"] --> SCATTER["scatter-add"] --> LN["LayerNorm+SiLU"] --> RES["x_atom += …"]
  end

  %% ---------- 输出 ----------
  subgraph OUT["⑤ 输出"]
    RES --> PRED["能量等 MLP"] --> GRAD["autograd → Force"]
  end

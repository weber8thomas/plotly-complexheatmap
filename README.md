# plotly-complexheatmap

**ComplexHeatmap-style visualizations built on Plotly** — clustered heatmaps with dendrograms, annotation tracks, and split heatmaps using native, interactive Plotly figures.

Inspired by the R [ComplexHeatmap](https://bioconductor.org/packages/ComplexHeatmap/) package by Zuguang Gu, this library brings the same expressive annotation and clustering features to the Python/Plotly ecosystem with **WebGL-first rendering** for large matrices.

## Installation

```bash
pip install plotly-complexheatmap

# Optional: faster clustering for large matrices
pip install plotly-complexheatmap[fast]
```

## Quick Start

```python
import numpy as np
import pandas as pd
from plotly_complexheatmap import ComplexHeatmap, HeatmapAnnotation

# Create a gene-expression matrix
rng = np.random.default_rng(42)
df = pd.DataFrame(
    rng.standard_normal((30, 12)),
    index=[f"gene_{i}" for i in range(30)],
    columns=[f"sample_{j}" for j in range(12)],
)

# Column annotations
top_ha = HeatmapAnnotation(
    group=["Control"] * 6 + ["Treatment"] * 6,
    quality=rng.uniform(0.7, 1.0, 12),
)

# Row annotations
right_ha = HeatmapAnnotation(
    pathway=["Metabolism"] * 10 + ["Signaling"] * 10 + ["Other"] * 10,
    which="row",
)

# Build the heatmap
hm = ComplexHeatmap(
    df,
    top_annotation=top_ha,
    right_annotation=right_ha,
    colorscale="RdBu_r",
    normalize="row",
    name="z-score",
)
fig = hm.to_plotly()
fig.show()
```

## Features

### Hierarchical Clustering

- Row and/or column clustering via `scipy.cluster.hierarchy`
- Configurable method (`ward`, `complete`, `average`, …) and metric (`euclidean`, `correlation`, …)
- Optional `fastcluster` backend for large matrices
- Dendrograms rendered as native Plotly line traces

### Annotation Tracks

Pass data as keyword arguments to `HeatmapAnnotation` — track type is auto-detected:

| Data Type | Rendered As | Example |
|-----------|------------|---------|
| String / categorical | Discrete colour bar | `group=["A", "B", "A"]` |
| Numeric array | Bar chart | `score=[0.5, 1.2, 0.8]` |
| Numeric + `type="scatter"` | Scatter points | `{"values": [...], "type": "scatter"}` |

Supported positions: `top_annotation` (column annotations) and `right_annotation` (row annotations).

### Split Heatmaps

Divide rows into groups that are clustered independently:

```python
hm = ComplexHeatmap(
    df,
    split_rows_by=["GroupA"] * 15 + ["GroupB"] * 15,
    # Or reference an annotation track by name:
    # split_rows_by="pathway",
)
```

### WebGL Rendering

For matrices larger than 100,000 cells, `go.Heatmapgl` (WebGL) is used automatically. Override with `use_webgl=True/False`.

### Z-Score Normalisation

```python
hm = ComplexHeatmap(df, normalize="row")    # per-row
hm = ComplexHeatmap(df, normalize="column") # per-column
hm = ComplexHeatmap(df, normalize="global") # whole matrix
```

## API Reference

### `ComplexHeatmap`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `data` | DataFrame / array | required | 2-D numeric data |
| `cluster_rows` | bool | `True` | Cluster rows |
| `cluster_cols` | bool | `True` | Cluster columns |
| `top_annotation` | `HeatmapAnnotation` | `None` | Column annotations |
| `right_annotation` | `HeatmapAnnotation` | `None` | Row annotations |
| `colorscale` | str / list | `"RdBu_r"` | Plotly colorscale |
| `normalize` | str | `"none"` | `"none"`, `"row"`, `"column"`, `"global"` |
| `use_webgl` | bool / None | `None` | Force WebGL on/off (auto by default) |
| `split_rows_by` | list / str | `None` | Row-split groups |
| `cluster_method` | str | `"ward"` | Linkage method |
| `cluster_metric` | str | `"euclidean"` | Distance metric |
| `name` | str | `""` | Colourbar title |
| `width` / `height` | int | 900 / 700 | Figure size in pixels |

### `HeatmapAnnotation`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `which` | str | `"column"` | `"column"` or `"row"` |
| `gap` | float | `0.005` | Gap between tracks (fraction) |
| `**kwargs` | various | — | Named annotation tracks |

## Dependencies

- `plotly >= 5.5.0`
- `scipy >= 1.9.0`
- `numpy >= 1.22.0`
- `pandas >= 1.4.0`
- Optional: `fastcluster >= 1.2.0`

## License

MIT

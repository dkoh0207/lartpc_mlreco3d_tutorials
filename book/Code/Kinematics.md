---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.10.3
kernelspec:
  display_name: Python 3
  language: python
  name: python3
execution:
  timeout: 60
---

# Kinematics and particle hierarchy

## Imports and configuration
If needed, you can edit the path to `lartpc_mlreco3d` library and to the data folder.
```{code-cell}
import os
SOFTWARE_DIR = '%s/lartpc_mlreco3d' % os.environ.get('HOME') 
DATA_DIR = os.environ.get('DATA_DIR')
```

The usual imports and setting the right `PYTHON_PATH`...  click if you need to see them.
```{code-cell}
:tags: [hide-cell]

import sys, os
# set software directory
sys.path.insert(0, SOFTWARE_DIR)
```

```{code-cell}
:tags: [hide-cell]

import numpy as np
import yaml
import torch
import plotly
import plotly.graph_objs as go
from plotly.offline import iplot, init_notebook_mode
init_notebook_mode(connected=False)

from mlreco.visualization import scatter_points, plotly_layout3d
from mlreco.visualization.gnn import scatter_clusters, network_topology, network_schematic
from mlreco.utils.ppn import uresnet_ppn_type_point_selector
from mlreco.utils.cluster.dense_cluster import fit_predict_np, gaussian_kernel
from mlreco.main_funcs import process_config, prepare
from mlreco.utils.gnn.cluster import get_cluster_label
from mlreco.utils.deghosting import adapt_labels_numpy as adapt_labels
from mlreco.visualization.gnn import network_topology

from larcv import larcv
```

The configuration is loaded from the file [inference.cfg](../data/inference.cfg).
```{code-cell}
:tags: [hide-output]

cfg=yaml.load(open('%s/inference.cfg' % DATA_DIR, 'r').read().replace('DATA_DIR', DATA_DIR),Loader=yaml.Loader)
# pre-process configuration (checks + certain non-specified default settings)
process_config(cfg)
# prepare function configures necessary "handlers"
hs=prepare(cfg)
```

The output is hidden because it reprints the entire (lengthy) configuration. Feel 
free to take a look if you are curious!

Finally we run the chain for 1 iteration:
```{code-cell}
# Call forward to run the net, store the output in "res"
data, output = hs.trainer.forward(hs.data_io_iter)
```
Now we can play with `data` and `output` to visualize what we are interested in. Feel free to change the
entry index if you want to look at a different entry!
```{code-cell}
entry = 0
```
Let us grab the interesting quantities:
```{code-cell}
clust_label = data['cluster_label'][entry]
input_data = data['input_data'][entry]
segment_label = data['segment_label'][entry][:, -1]

ghost_mask = output['ghost'][entry].argmax(axis=1) == 0
segment_pred = output['segmentation'][entry].argmax(axis=1)
```
## Particle hierarchy
## Momentum prediction
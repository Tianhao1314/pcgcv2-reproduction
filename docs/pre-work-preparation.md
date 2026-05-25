*Translated from the author's original Chinese study notes.*

# PCGCv2 Pre-Work Preparation: Paper Highlights + Pitfall Checklist

> Purpose: read this before the cloud GPU is rented. Afterwards you will know what this method does, what you need to produce in the end, and which pitfalls you are most likely to hit.
> Companion documents: *PCGCv2 One-Week Reproduction Plan*, *PCGCv2 Glossary of Terms and Software*
> Compiled: 2026-05-20

---

## A. Paper highlights (DCC 2021)

Paper: *Multiscale Point Cloud Geometry Compression*, Jianqiang Wang, Dandan Ding, Zhu Li, Zhan Ma. arXiv:2011.03799.

### A1. What problem it solves

A point cloud is a sparse, unordered, high-precision set of 3D points; storing/transmitting it directly is very costly. The difficulty is that points have no regular arrangement between them (unlike an image, which is a regular grid), so traditional compression methods cannot be applied directly. The paper's goal is to use deep learning to efficiently compress the **geometry information** of a point cloud (the positions of the points).

### A2. Core method (one sentence + breakdown)

One sentence: **use a sparse convolutional autoencoder to compress point cloud geometry in a multiscale, end-to-end manner**.

Broken down into four key points:

1. **Multiscale + progressive resampling**: it does not compress in one shot; instead it progressively downsamples the point cloud into coarser and coarser versions. On reconstruction it goes the other way, using the network's "learned resampling" to rebuild from coarse to fine, level by level.

2. **Sparse convolutional autoencoder**: the network body is an encoder (analyzes and aggregates features) + a decoder (reconstructs). Because point clouds are sparse, sparse convolution (MinkowskiEngine) only computes at locations that "have points", saving time and memory.

3. **Clever conversion at the bottleneck layer**: the input point cloud has only a single binary "occupancy" attribute (whether there is a point somewhere or not). At the bottleneck layer the network converts it into a **smaller point cloud**, and this small point cloud carries both a "geometry (position)" part and a "feature attribute" part.

4. **Hybrid coding**:
   - The **geometry (occupancy)** of that small point cloud at the bottleneck → **losslessly compressed** with an octree codec (i.e. tmc3), taking up very few bits;
   - The corresponding **feature attributes** → **lossily compressed** with a "learned probability context model" (this part is realized into a bitstream using the arithmetic coding library torchac).

### A3. Experimental results (the key numbers reported in the paper)

Compared with the two MPEG-standardized schemes:
- Versus **G-PCC** (Geometry-based Point Cloud Compression standard): BD-Rate reduced by **more than 70%**;
- Versus **V-PCC** (Video-based Point Cloud Compression standard): BD-Rate reduced by **more than 40%**;
- Encoding speed is comparable to G-PCC, and only about **1.5%** of V-PCC's (i.e. much faster).

> **BD-Rate (Bjontegaard Delta Rate)**: a metric that measures, between two rate-distortion curves, "how much bitrate is saved on average at equal quality". When the value is negative ("reduced by 70%"), it means a large improvement. This is the most commonly used summary number for comparing two methods in the point cloud / video compression field.

### A4. What this means for your reproduction

- The "result" you need to reproduce is essentially the **rate-distortion curve (RD curve) on the 8iVFB dataset**: bpp on the x-axis, D1/D2 PSNR on the y-axis.
- The paper has a figure of this curve. Using the pretrained models (r1–r7, 7 bitrate levels in total), you run each one on longdress / loot / redandblack / soldier, getting one (bpp, PSNR) point per level; connecting them gives your own curve.
- The repo's `./results` directory holds the results the authors officially produced, which you can use directly as your reference baseline — if your numbers match it, your environment and workflow are fine.
- Note: what you are reproducing is **PCGCv2's own curve**; you do not need to run G-PCC / V-PCC. The comparison against them in the paper was done by the authors; you just need to reproduce the PCGCv2 curve.

---

## B. Pitfall checklist (from the Issues of PCGCv2 and MinkowskiEngine)

### B1. Installing MinkowskiEngine (the most frequent and the hardest; reserve a full day)

| Pitfall | Symptom | Remedy |
|----|------|------|
| CUDA version mismatch | `The detected CUDA version mismatches the version used to compile PyTorch` | Make sure to use the PyTorch build with cu111; have `CUDA_HOME` point to cuda-11.1. Choosing the right AutoDL image basically avoids this |
| CUDA version too new | Compilation reports errors such as thrust namespace issues | Do not use CUDA 12.x; this project uses 11.1, do not exceed 11.8 |
| GCC/G++ version too high | Compilation fails partway through | Use GCC 7–9; if necessary `apt install gcc-9 g++-9`, then `export CC=gcc-9 CXX=g++-9` |
| openblas missing | Compilation cannot find blas | First run `apt install -y libopenblas-dev` |
| ninja version mismatch | Compilation reports ninja-related errors | `pip install -U ninja` |
| Compilation hangs / out of memory | The compilation process stalls or gets killed | `export MAX_JOBS=2` to limit parallelism |
| pip install method no longer works | `--install-option` has been removed from newer pip | Build from source: `python setup.py install --blas=openblas --force_cuda` |

### B2. tmc3 (the octree lossless encoding tool)

| Pitfall | Symptom | Remedy |
|----|------|------|
| **tmc3 outputs binary PLY** (Issue #4, frequent) | `UnicodeDecodeError: 'utf-8' codec can't decode byte 0x80` — because newer tmc3 outputs binary PLY by default, while PCGCv2 can only read ASCII PLY | When calling tmc3, add the parameter `--outputBinaryPly=0`. The current repo code may already include this fix; if you still get the error with a newer tmc3, check whether the tmc3 call in the code carries this parameter |
| **tmc3 lacks permissions** (Issue #5) | `/bin/sh: ...tmc3: Permission denied`, followed by `FileNotFoundError: ./output/xxx.ply` | `chmod 777 tmc3 pc_error_d` must be applied to **the actual tmc3 file being called**; confirm the tmc3 path configured in the code matches where you placed it. If tmc3 fails to run → the subsequent ply file is not generated → only then does FileNotFound appear; the root cause is tmc3 |

### B3. PyTorch version (Issue #9)

| Pitfall | Symptom | Remedy |
|----|------|------|
| PyTorch version too new | `TypeError: unsupported operand type(s) for //: 'Tensor' and 'int'` (the `//` integer division in coder.py errors) | Honestly use **PyTorch 1.8**, not 1.9+. This is exactly why the plan specifies 1.8.1 |

### B4. Running test.py (Issue #7)

| Pitfall | Symptom | Remedy |
|----|------|------|
| pc_error output column name does not match | `KeyError: 'mseF,PSNR(p2point)'` — the code reads the value by `mseF,PSNR(p2point)`, but pc_error's actual output may have a space: `mseF,PSNR (p2point)` | This is an inconsistency between pc_error's output format and the code's parsing. When you hit it, paste the error and pc_error's actual output to me, and we just adjust the column name being parsed |

### B5. Training-related (Issues #2 / #3, only encountered if you do the optional Day 7)

| Pitfall | Symptom | Remedy |
|----|------|------|
| Training data is empty | `IndexError: pop from empty list` (data_loader.py) | The training dataset path is wrong or the directory is empty; `--dataset` must point to the correct training data root directory |

### B6. Data download (Issue #6)

An old Issue mentioned that the test data link gave "permission denied". Just **use the latest link in the current README** (the box.nju.edu.cn direct link, or Baidu Netdisk, with extraction code `pcgc` in both cases). Downloading the box.nju.edu.cn direct link is fast on the AutoDL server.

---

## C. Quick reference for the three scripts' parameters

| Script | Function | Key parameters |
|------|------|----------|
| `coder.py` | Encode + decode a single point cloud | `--filedir` input point cloud, `--ckptdir` pretrained model, `--scaling_factor`, `--rho`, `--res` |
| `test.py` | Test and output bpp / D1 / D2 PSNR (iterates over multiple bitrate models) | `--filedir`, `--scaling_factor`, `--rho`, `--res` |
| `train.py` | Train from scratch | `--dataset` training dataset root directory |

Parameter meanings:

- **`--filedir`**: the input .ply point cloud file.
- **`--ckptdir`**: the pretrained model weights file (e.g. `ckpts/r3_0.10bpp.pth`).
- **`--scaling_factor`**: the scaling factor. Very high resolution point clouds (vox12) are scaled down before processing, which is why the README uses 0.375 for vox12 and 1.0 for vox10/vox11.
- **`--rho`**: a coefficient controlling the number/density of points in the decoded reconstruction. The larger rho is, the more points the reconstruction keeps.
- **`--res`**: the peak resolution of the point cloud, used to compute PSNR. The mapping: vox10 → 1024, vox11 → 2048, vox12 → 4096.

The standard test commands given in the README (the parameters differ for each point cloud — just copy them):

```
python test.py --filedir='longdress_vox10_1300.ply' --scaling_factor=1.0 --rho=1.0 --res=1024
python test.py --filedir='dancer_vox11_00000001.ply' --scaling_factor=1.0 --rho=1.0 --res=2048
python test.py --filedir='Staue_Klimt_vox12.ply' --scaling_factor=0.375 --rho=4.0 --res=4096
python test.py --filedir='House_without_roof_00057_vox12.ply' --scaling_factor=0.375 --rho=1.0 --res=4096
```

---

## D. Pre-work self-check

Once the machine is rented and connected, start from Day 1 of the *One-Week Reproduction Plan*. For any of the errors above, first self-check against this checklist; if you cannot pin it down, paste the **full error message** to me and I will help you debug it.

The thing most likely to eat time is B1 (MinkowskiEngine) — once that is installed and working, the rest basically goes smoothly.

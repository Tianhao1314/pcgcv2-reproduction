*Translated from the author's original Chinese study notes.*

# PCGCv2 One-Week Reproduction Plan

> Project: NJUVISION/PCGCv2 — Multiscale Point Cloud Geometry Compression (DCC 2021)
> Repository: https://github.com/NJUVISION/PCGCv2
> Plan date: 2026-05-20 | Total deadline given by advisor: 2 weeks
> This plan: get it running + reproduce the metrics in 7 working days, leaving about 1 week as buffer / optional items

---

## 0. First, be clear: what counts as "a successful reproduction"

Reproduction comes in three tiers, and what the advisor means by "reproduction" is almost certainly the second tier:

| Tier | Content | Goal in this plan |
|------|------|-----------|
| Tier 1: Get inference running | Environment set up; successfully encode + decode a single point cloud with the pretrained model, with no errors | Complete by Day 3 |
| **Tier 2: Reproduce the paper's metrics** | On the 8iVFB dataset used in the paper, produce bpp / D1 PSNR / D2 PSNR, draw the rate-distortion curve (RD curve), and confirm it basically matches the paper's figures | **Complete by Day 6 ← the bar for handing it in** |
| Tier 3: Train from scratch | Retrain the model yourself with train.py and reach the paper's metrics | Optional, only if time allows |

**Final deliverable**: a reproduction report containing ① the RD curve you produced + a comparison against the paper's curve; ② a table of key metric values; ③ environment configuration notes + pitfalls encountered. Deliver this and, as far as the advisor is concerned, the reproduction is done.

---

## 1. Key prerequisite: you must rent a cloud GPU

PCGCv2 depends on **MinkowskiEngine** (a sparse convolution library), which **strictly requires NVIDIA CUDA**. A Mac (Intel or Apple silicon, it makes no difference) has no CUDA, so **it cannot run locally**.

**Plan: rent an AutoDL cloud GPU.** Your Mac only does two things — SSH into the machine and view results in a browser.

- Recommended spec: **RTX 3090 (24 GB VRAM)**, about ¥1.5–2.5/hour
- Estimated total cost: about 30–40 hours of active use → **roughly ¥80–120** (remember to "shut down" to stop GPU billing when not in use)
- Registration: https://www.autodl.com/ (account registration requires real-name verification)

> Tip: After an AutoDL instance is "shut down", the data disk is preserved, so the environment is still there next time you boot it. Billing only happens while the instance is running. Shut it down every day after you finish.

---

## 2. Environment version combination (the combination least likely to cause trouble — copy it exactly)

MinkowskiEngine is extremely sensitive to versions, so **do not use the latest CUDA version**. From research, the most stable combination is:

| Component | Version | Notes |
|------|------|------|
| Python | 3.8 | |
| CUDA | 11.1 | Do not use 11.8 / 12.x; MinkowskiEngine will fail to compile |
| PyTorch | 1.8.1 + cu111 | |
| MinkowskiEngine | 0.5.4 (built from source) | The hardest step |
| torchac | 0.9.3 | Install with pip |
| GCC/G++ | 7–9 (≥7.4) | Too high a version causes compilation to fail |
| tmc3 | v12 | MPEG tool, built with cmake |
| pc_error_d | bundled in the repo | Computes D1/D2 distortion; just chmod it |

**When renting the instance**: in the AutoDL instance creation screen, pick the base image `PyTorch 1.8.1 / Python 3.8 / CUDA 11.1` directly. That way PyTorch and CUDA are already configured, sparing you the step that most easily goes wrong.

---

## 3. Day-by-day plan

### Day 1 — Rent the machine + set up the basic environment + download materials (about 3 hours, half a day)

1. Register on AutoDL, top up about ¥50, and create an instance: RTX 3090 + the image `PyTorch 1.8.1 / Python 3.8 / CUDA 11.1`.
2. SSH into it from the Mac's "Terminal" (the AutoDL console gives you the login command).
3. Verify PyTorch and CUDA:
   ```bash
   python -c "import torch; print(torch.__version__, torch.cuda.is_available())"
   # Expected output similar to: 1.8.1+cu111 True
   nvcc --version   # Expected to show release 11.1
   ```
4. Clone the project code:
   ```bash
   cd ~/autodl-tmp        # data disk, plenty of space
   git clone https://github.com/NJUVISION/PCGCv2.git
   ```
5. Download the pretrained model + test data (the README gives Nanjing University network-drive direct links at box.nju.edu.cn, which can be downloaded directly with wget from within China; if that fails, use Baidu Netdisk — the password in both cases is pcgc):
   - Pretrained model: https://box.nju.edu.cn/f/b2bc67b7baab404ea68b/
   - Test data: https://box.nju.edu.cn/f/30b96fc6332646c5980a/
   - Put them in `PCGCv2/ckpts/` and `PCGCv2/testdata/` (use the exact directory names from the repo)

**Day 1 acceptance criteria**: `torch.cuda.is_available()` returns True, and the code, model, and data are all on the server.

---

### Day 2 — Install MinkowskiEngine (the hardest day; reserve a full day)

```bash
# 1. Install system dependencies
apt update && apt install -y libopenblas-dev build-essential
pip install ninja

# 2. Confirm the CUDA path
export CUDA_HOME=/usr/local/cuda
echo $CUDA_HOME

# 3. Build and install MinkowskiEngine 0.5.4 from source
cd ~/autodl-tmp
git clone https://github.com/NVIDIA/MinkowskiEngine.git
cd MinkowskiEngine
export MAX_JOBS=4      # limit parallel compilation to avoid running out of memory
python setup.py install --blas=openblas --force_cuda

# 4. Verify
python -c "import MinkowskiEngine as ME; print(ME.__version__)"
```

**Common pitfalls and remedies**:
- `CUDA version mismatch`: confirm you are using the PyTorch build with cu111; if necessary, check that `CUDA_HOME` points to cuda-11.1.
- Compilation fails because the GCC version is too high: `apt install gcc-9 g++-9`, then `export CC=gcc-9 CXX=g++-9` and retry.
- Compilation hangs / out of memory: lower `MAX_JOBS` to 2.
- Keep the full error messages; work through them one at a time rather than changing several things at once.

Then install torchac:
```bash
pip install torchac==0.9.3
python -c "import torchac; print('torchac ok')"
```

**Day 2 acceptance criteria**: both `import MinkowskiEngine` and `import torchac` run without errors.

---

### Day 3 — Build tmc3 + get the first encode/decode running (Tier 1 achieved)

```bash
# 1. Build tmc3 (the MPEG point cloud lossless encoding tool)
cd ~/autodl-tmp
git clone https://github.com/MPEGGroup/mpeg-pcc-tmc13.git
cd mpeg-pcc-tmc13
mkdir build && cd build
cmake .. && make -j4
# The built executable is at build/tmc3/tmc3; copy it into the PCGCv2 directory
cp tmc3/tmc3 ~/autodl-tmp/PCGCv2/

# 2. Give the bundled tools execute permission
cd ~/autodl-tmp/PCGCv2
chmod 777 tmc3 pc_error_d

# 3. Run encode + decode on a single point cloud
python coder.py --filedir='longdress_vox10_1300.ply' --ckptdir='ckpts/r3_0.10bpp.pth' --scaling_factor=1.0 --rho=1.0 --res=1024
```

> If the `mpeg-pcc-tmc13` repository is inaccessible (the MPEG repo occasionally requires permissions), fall back to a mirror source or another way of obtaining tmc3 v12.

**Day 3 acceptance criteria (= Tier 1 "get inference running" complete)**: a compressed bitstream file and a decoded point cloud are successfully generated, with no errors throughout.

---

### Day 4 — Get test.py running, and understand the output metrics

```bash
python test.py --filedir='longdress_vox10_1300.ply' --scaling_factor=1.0 --rho=1.0 --res=1024
```
- Understand the **bpp** (bits per point — lower means more aggressive compression), **D1 PSNR** (point-to-point distortion), and **D2 PSNR** (point-to-plane distortion) in the output.
- Understand that the r1–r7 models in `ckpts/` are different bitrate levels; one model corresponds to one point on the RD curve.
- Try running a point cloud at another resolution (e.g. dancer_vox11 from the README) to confirm what the `--res` / `--scaling_factor` parameters do.

**Day 4 acceptance criteria**: you can clearly explain what every number in a test output means.

---

### Day 5 — Batch testing across the whole 8iVFB dataset × multiple bitrates

- For the 4 sequences of 8iVFB (longdress / loot / redandblack / soldier) × multiple bitrate models (r1–r7), batch-run test.py.
- Write a batch script that iterates over all "sequence × model" combinations and collects bpp / D1 / D2 into a CSV.
- The `./results` directory bundled in the repo holds the authors' official test results, which can serve as a reference baseline.

**Day 5 acceptance criteria**: a complete results table (CSV) covering all sequences and bitrate levels.

---

### Day 6 — Draw the RD curve + compare against the paper

- With Python (matplotlib), turn the CSV into a rate-distortion curve: bpp on the x-axis, D1/D2 PSNR on the y-axis, one curve per sequence.
- Compare it against the RD curve figures in the DCC2021 paper to see whether it basically matches (small deviations are normal).
- Write the plotting script from that CSV; one curve per sequence, seven rate points each.

**Day 6 acceptance criteria (= Tier 2 "reproduce the paper's metrics" complete ← the bar for handing it in)**: your curve basically overlaps the paper's curve.

---

### Day 7 — Write the reproduction report + buffer

The reproduction report should include:
1. Project overview + reproduction goal
2. Environment configuration (version table + key commands)
3. Reproduction results: RD curve comparison figure + metrics table
4. Problems encountered and how they were solved (especially MinkowskiEngine)
5. Conclusion: was the reproduction successful

The finished report is in `report/` (English and Chinese, Word and PDF).

---

## 4. How to use the remaining roughly 1 week of buffer

- **First choice**: keep it as buffer in case any step in Days 1–7 overruns (MinkowskiEngine often eats up time).
- **If progress is smooth (optional, bonus)**: try Tier 3 — run a from-scratch training pass with `train.py` to experience the full training workflow:
  ```bash
  python train.py --dataset='training_dataset_rootdir'
  ```
  Training dataset: https://box.nju.edu.cn/f/1ff01798d4fc47908ac8/
  Note: a full training run takes a long time (several GPU-days); you can train only a few epochs just to verify the workflow runs.

---

## 5. Risk overview (sorted by probability)

| Risk | Probability | Response |
|------|------|------|
| MinkowskiEngine fails to compile | High | Follow the version combination strictly; reserve a full Day 2 for it |
| tmc3 repo inaccessible / fails to compile | Medium | Switch to a mirror, or get the v12 binary another way |
| Slow network-drive downloads / dead links | Medium | Prefer the box.nju.edu.cn direct links; fall back to Baidu Netdisk (password pcgc) |
| Cloud GPU cost exceeds expectations | Low | Shut down immediately when not in use; a 3090 is enough, no need for a pricier card |
| Reproduced curve deviates a lot from the paper | Low | Check that the --res / --scaling_factor / --rho parameters are set correctly for each point cloud per the README |

---

## 6. Execution notes

- For any error at any step, read the traceback line by line before changing anything — most of these failures are version mismatches, not logic bugs.
- Day 5 needs a batch testing script and Day 6 an RD plotting script; both are small and worth writing before the GPU clock starts.
- Day 7: write up the reproduction report, with the RD tables and the comparison against the official results.
- Account registration, top-up and real-name verification all have to be done in person. Real-name verification is the step that failed from outside China, which is why this reproduction ended up on RunPod rather than AutoDL (see the README).

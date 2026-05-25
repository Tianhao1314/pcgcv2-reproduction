*Translated from the author's original Chinese study notes.*

# PCGCv2 Reproduction Project · Glossary of Terms and Software

Multiscale Point Cloud Geometry Compression

Compiled 2026-05-20 | To be used together with *PCGCv2 One-Week Reproduction Plan*

This quick-reference glossary explains, in plain language and grouped by topic, all the technical terms and software you will encounter while reproducing PCGCv2. It is recommended that you keep it at hand while running the project.

## 1. Basic concepts: what a point cloud is

**Point cloud** — A data form that represents an object or a scene with a large number of three-dimensional coordinate points (x, y, z). What you get from a LiDAR scan or a 3D scanner is a point cloud — thousands, even millions, of isolated points. It carries no "surfaces" of its own, and there is no connectivity relationship between the points; it is just a cloud of points.

**Point cloud geometry and point cloud attributes (geometry & attribute)** — A point cloud's information falls into two kinds: "geometry" refers to each point's position (where it is); "attributes" refer to the color, normal vector, reflection intensity, etc. attached to each point. The PCGCv2 project compresses only the "geometry"; it does not handle attributes such as color.

**Voxel** — Cutting three-dimensional space into a regular array of small cubic cells, the equivalent of a 3D version of the "pixel". The vox10 / vox11 / vox12 in the test file names mean the coordinates are quantized to a precision of 2 to the 10th, 11th, or 12th power; the larger the number, the finer the space and the more points.

**.ply file** — A common point cloud / 3D model file format that stores each point's coordinates (and possibly its color) line by line. The test point clouds in this project are all in .ply format, for example longdress_vox10_1300.ply.

**Mesh (a point of contrast)** — Another way of representing 3D, made up of vertices plus triangular faces, with explicit connectivity. A point cloud has no "surfaces" — this is the biggest difference between it and a mesh, and it is precisely because there is no connectivity that point cloud compression needs dedicated methods.

## 2. Compression-related concepts

**Codec (encode / decode)** — A complete set of methods for compressing data (encoding) and then restoring it (decoding). What coder.py in the project does is exactly this.

**Lossless compression** — After decompression the data is exactly the same as the original, with no information lost. PCGCv2 uses lossless compression internally for the "point occupancy information".

**Lossy compression** — After decompression the data is approximate, with some details discarded in exchange for a higher compression ratio (a smaller file). PCGCv2 as a whole is a lossy compression scheme.

**Bitrate / bpp (bits per point)** — The unit for measuring the degree of compression: how many bits are used on average to store each point. The lower the bpp, the more aggressive the compression and the smaller the file. The pretrained model name r3_0.10bpp means this model compresses to roughly 0.10 bpp.

**Rate-distortion curve / RD curve (rate-distortion curve)** — The most central evaluation chart in the compression field. The x-axis is the bitrate (bpp), the y-axis is the distortion or quality (PSNR). A model produces one point at one bitrate level, and multiple points connect into a curve. The core goal of this reproduction is exactly to draw this curve and compare it against the curve in the paper.

**D1 PSNR (point-to-point distortion)** — One of the metrics for measuring reconstructed point cloud quality; it compares the "point-to-point" distance error between reconstructed points and original points. The higher the PSNR value, the better the quality and the smaller the distortion.

**D2 PSNR (point-to-plane distortion)** — Another quality metric, measuring the distance from a reconstructed point to the original surface; it is closer to the perception of the human eye than D1. The pc_error_d tool bundled with the project is used to compute D1 / D2.

**Entropy coding / arithmetic coding (entropy coding / arithmetic coding)** — A technique that encodes data into a bitstream "close to the theoretical minimum length" according to the probability of occurrence. Content with a high probability of occurrence uses fewer bits. What the torchac library does is exactly arithmetic coding.

## 3. Deep-learning-related concepts

**Neural network** — A mathematical model that can "learn"; after being trained on a large amount of data, it can perform a specific task (here, compressing and reconstructing point clouds).

**End-to-end learning** — The whole "compression → restoration" workflow is trained and optimized in a unified way by a single neural network: input the original point cloud, output the reconstructed point cloud, with everything in between learned together, rather than having a human stitch the modules together.

**Autoencoder** — A network with an "encoder + decoder" structure. The encoder compresses the data into a compact intermediate representation, and the decoder restores the data from this intermediate representation — naturally suited to compression tasks.

**Encoder / decoder** — The two halves of an autoencoder. The encoder is responsible for "compressing", turning the point cloud into a compact representation; the decoder is responsible for "restoring", reconstructing the compact representation back into a point cloud.

**Bottleneck layer** — The "narrowest" layer in the middle of an autoencoder, where the information is compressed to its smallest. What is actually stored or transmitted after compression is the data of this layer.

**Convolution** — The basic operation used in a neural network to extract local features; it is widely used in image and 3D data processing.

**Sparsity** — In the three-dimensional space a point cloud occupies, the vast majority of cells are empty, and only a few locations have points. This property of "mostly empty" is called sparsity.

**Sparse convolution** — Ordinary convolution computes once for every cell (including empty ones), which is very wasteful; sparse convolution only does the computation at locations that "have points", making it fast and memory-efficient for point clouds. PCGCv2's network is built on sparse convolution.

**Down-sampling / up-sampling / re-sampling (down- / up- / re-sampling)** — Downsampling = making the points fewer and coarser; upsampling = adding points back, making them finer; resampling is a general term for both kinds of operation. When PCGCv2 restores a point cloud, it uses the network's "learned resampling" to reconstruct a coarse point cloud into a fine one level by level.

**Multiscale** — Organizing a point cloud into multiple levels from "coarse to fine" and processing it level by level. This is exactly what "Multiscale" in the PCGCv2 name means, and it is its core idea.

**Occupancy information (occupancy)** — Binary (0 / 1) information that describes "whether a given cell actually has a point in it". At the bottleneck layer, it is losslessly coded with very few bits.

**Pretrained model** — A model weights file (with extension .pth) that the authors have already trained and made available for direct download and use. With it, you can do compression tests directly without training one yourself.

**Training / Epoch / dataset (training / epoch / dataset)** — The process of repeatedly adjusting the network's parameters with a large amount of data is called "training"; going through the entire dataset once completely is called one epoch; the "training dataset" is the batch of point clouds used for training. What train.py in the project does is exactly training.

## 4. Software, tools, and platforms

**PCGCv2** — The project itself that you are reproducing this time, an open-source point cloud geometry compression codebase developed by Nanjing University's Vision Lab (NJU Vision Lab), corresponding to a DCC 2021 paper.

**SparsePCGC** — An upgraded unified framework the authors released after PCGCv2; it supports lossless / lossy compression and both dense point clouds and sparse LiDAR point clouds. It is not reproduced this time, but is worth knowing about.

**MinkowskiEngine** — NVIDIA's open-source sparse convolution library, on which PCGCv2's neural network is built. It strictly depends on NVIDIA's CUDA, and is the hardest part of the whole environment configuration to install and the most prone to errors.

**torchac** — A Python library for arithmetic coding. PCGCv2 uses it to encode the data output by the neural network into the final compressed bitstream.

**tmc3** — The official point cloud compression tool of MPEG (the organization that sets international compression standards for audio, video, and point clouds). PCGCv2 uses it to losslessly compress the "coarse coordinates after downsampling". It needs to be built with cmake yourself.

**pc_error_d** — A small command-line tool for computing the D1 / D2 distortion metrics, bundled in the PCGCv2 repo. Before use, just add execute permission with the chmod command.

**PyTorch** — Currently one of the most mainstream deep learning frameworks; PCGCv2's code is written in it. Its version must correspond to the CUDA version (this project uses 1.8.1 + cu111).

**CUDA / NVIDIA GPU** — CUDA is the platform NVIDIA graphics cards use for parallel computing. Deep learning almost always relies on NVIDIA GPUs for acceleration. Apple computers (Macs) have no CUDA — this is the fundamental reason you must use a machine with an NVIDIA graphics card.

**Python / conda** — Python is the programming language used in this project; conda is a tool for managing Python environments and dependency versions, able to isolate independent, non-interfering environments for different projects.

**SSH** — A remote login protocol. You use the Mac's built-in "Terminal" to connect to the remote GPU server via SSH, and type commands on it to operate.

**Sources of GPU compute: cloud GPUs and HPC clusters** — Running this project requires an NVIDIA GPU. There are usually two routes: ① a commercial cloud GPU platform (such as AutoDL), pay-by-the-hour, ready to use immediately, with root privileges; ② a university's high-performance computing (HPC) cluster — free but requiring an application and queueing. For FAU students, the second route is recommended first.

**NHR@FAU / Alex cluster** — The Erlangen National High Performance Computing Center (NHR@FAU) operated by FAU itself. Its Alex cluster is equipped with NVIDIA A100 / A40 GPUs (more than 600 of them). FAU students can apply to use it, for free — it is the most cost-effective source of compute for running this project.

**HPC cluster (High Performance Computing)** — A shared supercomputing system made up of a large number of compute nodes. Unlike a cloud server: you generally do not have root privileges (you have to install software in user space with conda), and jobs must be submitted to a job scheduling system to queue.

**SLURM** — The job scheduling system on HPC clusters. You write the task you want to run as a script and submit it to SLURM, which is responsible for queueing and allocating a GPU to you to run on.

**module (environment modules)** — A mechanism on HPC for loading a specified software version (such as a particular version of CUDA, Python, or a compiler), with commands of the form `module load cuda/11.x`.

**AutoDL** — A Chinese commercial cloud GPU rental platform, billed by the hour. If FAU's HPC application does not come through in time, it can serve as an emergency fallback; but for FAU students, the on-campus HPC is usually the better first choice.

**8iVFB dataset** — The standard test point cloud dataset used in the paper (8i Voxelized Full Bodies), containing human-body point cloud sequences such as longdress, loot, redandblack, and soldier. The metrics for the reproduction are measured on this dataset.

**DCC 2021** — Data Compression Conference 2021, an international academic conference in the data compression field. The PCGCv2 paper was published here.

## 5. Tying it all together in one sentence

PCGCv2 is a project that uses a "sparse convolutional autoencoder" to compress point cloud geometry in an "end-to-end" manner.

When encoding: the input .ply point cloud is downsampled level by level through a multiscale structure; at the bottleneck layer, the "occupancy information" is losslessly compressed with arithmetic coding (torchac) and the features are lossily compressed; the coordinates of the coarsest level are handed to tmc3 for lossless compression.

When decoding: it is upsampled level by level and reconstructed back into a complete point cloud.

And "reproduction", as a task, is: on a machine with an NVIDIA GPU (FAU's NHR@FAU cluster, or a cloud GPU), set up the PyTorch + CUDA + MinkowskiEngine environment, use the pretrained models to produce bpp and D1 / D2 PSNR on the 8iVFB dataset, draw them as a rate-distortion curve, and compare it against the curve in the paper — if the curve basically matches, the reproduction counts as successful.

Companion file: PCGCv2 One-Week Reproduction Plan.md (day-by-day operation steps and environment configuration commands)
Project repository: github.com/NJUVISION/PCGCv2

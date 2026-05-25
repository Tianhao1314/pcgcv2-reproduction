*Translated from the author's original Chinese study notes.*

# PCGCv2 Glossary of Terms · Vivid-Example Edition

Each concept comes with an everyday example, to help you understand and remember it quickly.

Below, each term is first given an explanation, then a vivid analogy marked with an orange "*Example:*". The analogies are only meant to aid understanding; they do not aim for 100% rigor.

## 1. Basic concepts: what a point cloud is

**Point cloud** — A data form that represents an object or a scene with a large number of three-dimensional coordinate points (x, y, z); it carries no "surfaces" of its own and no connectivity relationship between the points.
*Example:* Imagine sticking hundreds of thousands of glowing little beads all over a person, each bead recording its own spatial coordinates; then "switch the person off" and leave only this pile of glowing beads suspended in the air — that pile of beads is the point cloud.

**Point cloud geometry and attributes (geometry & attribute)** — Geometry refers to each point's position; attributes refer to the color, normal vector, etc. attached to a point. PCGCv2 compresses only the geometry.
*Example:* Geometry = "at which position in space each bead hangs"; attributes = "what color each bead is, which direction it faces". PCGCv2 cares only about where the beads hang, not what color they are.

**Voxel** — Cutting three-dimensional space into a regular array of small cubic cells, the equivalent of a 3D version of the pixel.
*Example:* Imagine the whole room as a LEGO baseplate extended into three dimensions — space is cut into LEGO cells, each either "has a brick" or "is empty". vox10 is cutting it into LEGO cells as fine as 1024×1024×1024.

**.ply file** — A common point cloud file format that stores each point's coordinates line by line.
*Example:* Like an Excel sheet, with each row writing one point's x, y, z (and possibly its color); the .ply file is the "box" that holds this sheet.

**Mesh (a point of contrast)** — Another 3D representation made up of vertices plus triangular faces, with connectivity. A point cloud has no surfaces.
*Example:* A mesh is like "a clothing model sewn out of triangular fabric", with lines connecting point to point to form surfaces; a point cloud is like "shaking all the beads off that piece of clothing", leaving only beads scattered on the floor and no fabric.

## 2. Compression-related concepts

**Codec** — A complete set of methods for compressing data (encoding) and then restoring it (decoding).
*Example:* Like vacuum-compressing a down jacket into a small bag (encoding), and when you want to wear it, letting the air back in so it puffs up to its original shape (decoding).

**Lossless compression** — After decompression the data is exactly the same as the original, with no information lost.
*Example:* Like using ZIP to compress a contract document — what comes out of decompression is identical down to every character and punctuation mark.

**Lossy compression** — After decompression the data is approximate, with some details discarded in exchange for a higher compression ratio.
*Example:* Like saving a song as an MP3 — the file is much smaller, but compared with the studio's original master, some details that the human ear can barely perceive have been discarded.

**Bitrate / bpp (bits per point)** — How many bits are used on average to store each point; the lower, the more aggressive the compression.
*Example:* Like working out "how much rice each guest eats on average". The lower the bpp, the more economical the "storage ration" spent on each point.

**Rate-distortion curve / RD curve** — A trade-off curve with bitrate (bpp) on the x-axis and quality (PSNR) on the y-axis; it is the core goal of the reproduction.
*Example:* Like the "fuel consumption vs. horsepower" trade-off chart drawn when buying a car — do you want to save fuel (low bpp) or want strong power (high quality); the curve lays out the cost of each choice at a glance.

**D1 PSNR (point-to-point distortion)** — Compares the "point-to-point" distance error between reconstructed points and original points; the higher, the better.
*Example:* Like tracing characters in elementary school: overlay the characters you wrote on the template and measure stroke by stroke "how much this stroke is off". The less it is off, the higher the score.

**D2 PSNR (point-to-plane distortion)** — Measures the distance from a reconstructed point to the original "surface"; closer to the perception of the human eye than D1.
*Example:* Like checking wallpaper: what you look at is whether the wallpaper as a whole is hung flat against the wall, rather than aligning it with a specific nail on the wall.

**Entropy coding / arithmetic coding** — Encoding data into a bitstream close to the theoretical minimum length according to probability of occurrence.
*Example:* Like sending a telegram: give the most-used letters very short codes and rare letters long codes. In Morse code, the most common letter E uses just a single dot — that is the principle.

## 3. Deep-learning-related concepts

**Neural network** — A mathematical model that can "learn" from a large amount of data and afterwards perform a specific task.
*Example:* Like a child who has seen several million cat and dog photos and gradually "figures out" on their own how to recognize cats and dogs, rather than having someone recite rigid rules to them one by one.

**End-to-end learning** — The whole workflow is trained and optimized in a unified way by one network, rather than humans stitching the modules together.
*Example:* Like teaching someone to drive: instead of teaching "press the gas", "turn the wheel", "check the mirror" separately and then assembling them, you have them get straight into the car and practice driving as a whole from start to finish, with all actions coordinated together.

**Autoencoder** — A network with an "encoder + decoder" structure that first compresses into a compact representation, then restores.
*Example:* Like a stenographer: after hearing a whole passage of speech, condense it into a few key words written on a small slip of paper (encoding); afterwards, from these few key words, retell the whole passage (decoding).

**Encoder / decoder** — The two halves of an autoencoder, responsible respectively for "compressing" and "restoring".
*Example:* They are the two halves of the stenographer's skill — "jotting down key words while listening" is the encoder, "retelling while looking at the key words" is the decoder.

**Bottleneck layer** — The layer in the network where information is compressed to its smallest; what is actually stored after compression is this layer.
*Example:* It is the few key words on the stenographer's slip of paper themselves — all the information is condensed into the smallest place, and what really needs to be saved and passed on is this little slip.

**Convolution** — The basic operation in a neural network for extracting local features.
*Example:* Like sliding a small magnifying glass over a picture cell by cell, looking at each spot to see "is there an edge nearby, is there a pattern".

**Sparsity** — In three-dimensional space, the vast majority of cells are empty, and only a few have points.
*Example:* Like the night sky — almost the whole sky is black, and only a very few locations are dotted with stars. That is how "empty" a point cloud is in three-dimensional space.

**Sparse convolution** — Doing the computation only at locations that "have points", making it fast and memory-efficient for point clouds.
*Example:* Since most of the night sky is pitch black, the smart approach is to look only where there are stars, rather than carefully scanning every patch of pitch-black sky — sparse convolution is "looking only where there are stars".

**Down-sampling / up-sampling / re-sampling** — Downsampling makes the points fewer and coarser; upsampling adds points back and makes them finer.
*Example:* Downsampling is like shrinking a high-definition large image into a tiny thumbnail (fewer points); upsampling is like enlarging the thumbnail again and trying hard to add detail back (more points).

**Multiscale** — Organizing a point cloud into multiple levels from "coarse to fine" and processing it level by level.
*Example:* Like continually zooming in while looking at an electronic map: first look at the province-level map (coarse), then the city level, then the street level (fine), refining level by level.

**Occupancy information (occupancy)** — Binary (0/1) information that describes "whether a given cell actually has a point in it".
*Example:* Like the checkmark for "present / absent" against each seat on a classroom roll call sheet; it answers just one question: does this cell have a point in it.

**Pretrained model** — A model weights file that the authors have already trained and made available for direct download and use.
*Example:* Like a child that someone else has already raised for you and taught to recognize cats — you can just take them home and use them, without having to raise one from scratch yourself.

**Training / Epoch / dataset** — Repeatedly tuning parameters with a large amount of data is called training; going through the dataset once is called one epoch.
*Example:* Training is like preparing for an exam by doing practice problems; finishing a whole problem set from beginning to end once is called one epoch; the "dataset" is the problem set itself.

## 4. Software, tools, and platforms

**PCGCv2** — The project itself that you are reproducing this time, an open-source point cloud geometry compression codebase from Nanjing University.
*Example:* It is the recipe itself that you are going to "follow to remake this dish" this time.

**SparsePCGC** — An upgraded unified framework the authors released after PCGCv2.
*Example:* Like an "upgraded recipe" the same chef released later, with a fuller menu — it can even make that new dish, LiDAR point clouds.

**MinkowskiEngine** — NVIDIA's sparse convolution library, on which PCGCv2's network is built; the hardest to install.
*Example:* Like a high-end food processor specially for cutting vegetables: this dish cannot be made without it, but this very machine is the hardest to assemble — it is also the biggest hurdle on your reproduction journey.

**torchac** — A library for arithmetic coding that packs the network's output into a compressed bitstream.
*Example:* Like a sealing machine that "packs neatly-arranged items down tight and seals them into the smallest delivery box".

**tmc3** — MPEG's official point cloud compression tool, used for losslessly compressing coarse coordinates.
*Example:* Like a "veteran professional sealing machine" produced by MPEG; PCGCv2 specially calls on it to package up that small portion of the coarsest coordinate data.

**pc_error_d** — A small command-line tool for computing the D1 / D2 distortion metrics.
*Example:* Like a tape measure, specially used to measure "exactly how much the reconstructed point cloud differs from the original".

**PyTorch** — A mainstream deep learning framework, in which PCGCv2 is written.
*Example:* Like that "household-name kitchen-appliance brand" of cookware used for cooking; most deep learning recipes on the market are written based on it.

**CUDA / NVIDIA GPU** — NVIDIA graphics cards' parallel computing platform; deep learning almost always relies on it for acceleration.
*Example:* CUDA is the "stove" exclusive to NVIDIA graphics cards; the dish of deep learning can only be cooked on this kind of stove; and a Mac's kitchen simply does not have this kind of stove installed — that is why you have to rent a machine.

**Python / conda** — Python is the project's language; conda is used to isolate environments and dependency versions.
*Example:* Python is the language the recipe is written in; conda is like fitting out a separate, independent kitchen for each dish, each cooked on its own without cross-contamination of flavors.

**SSH** — A remote login protocol; you operate a remote server with a local terminal.
*Example:* Like operating a computer in another room with a remote control — you type commands on your own Mac, but they are actually executed on the distant cloud server.

**Sources of GPU compute: cloud GPUs and HPC** — Either rent a cloud GPU by the hour, or apply for a university's free HPC cluster.
*Example:* Either go to a "shared kitchen that charges by the hour" (cloud GPU, available whenever you arrive), or use the "free big-cafeteria back kitchen of the school" (HPC, powerful but requiring queueing and an application).

**NHR@FAU / Alex cluster** — FAU's own national-level HPC center; the Alex cluster is equipped with A100/A40 GPUs.
*Example:* It is the "huge free back kitchen" that your school FAU has of its own, with hundreds of top-spec stoves (A100 GPUs) in it, which students can apply to enter and cook in.

**HPC cluster** — A shared supercomputer made up of a large number of nodes; no root privileges, queueing required.
*Example:* Like a giant public kitchen shared by hundreds of people: you cannot remodel it as you please (no root privileges), and to use a stove you have to queue and take a number.

**SLURM** — The job scheduling system on HPC, responsible for queueing and allocating GPUs.
*Example:* It is that public kitchen's "number-calling system" — you write the dish you want to make on a slip and submit it, and the system queues and calls numbers, only letting you onto the stove when it is your turn.

**module (environment modules)** — A mechanism on HPC for loading a specified software version on demand.
*Example:* Like a row of "use-on-demand tool" lockers in the public kitchen; typing `module load cuda` is going to the locker to take out a specified version of CUDA to use.

**AutoDL** — A Chinese commercial cloud GPU platform, billed by the hour, WeChat payment.
*Example:* Like a chain store that "rents out kitchens by the hour", ready to use the moment you open the door, leave when you are done, pay by scanning a WeChat QR code.

**8iVFB dataset** — The standard test point cloud dataset used in the paper (human-body sequences).
*Example:* Like the "standard sample exam paper" used in a test — all researchers use this same set of human-body point clouds to score, so the results can be compared side by side.

**DCC 2021** — An international academic conference in the data compression field; the PCGCv2 paper was published here.
*Example:* Like the data compression circle's annual "industry conference"; the PCGCv2 paper made its official debut on this stage.

## 5. Tying the whole chain together with one example

Imagine PCGCv2's workflow as "packing a bead statue for shipping by mail":

This statue is made of hundreds of thousands of beads (the point cloud). Copying out the list of every bead's coordinates and shipping it directly takes up too much space. So PCGCv2, this "packing master", first shrinks the statue level by level into several rough models from coarse to fine (multiscale downsampling); on the smallest rough model (the bottleneck layer), it records "which cells have beads" as an extremely simple presence/absence list and seals it losslessly and snugly with the veteran sealing machine tmc3, and packs the detail features of each bead down tight with the sealing machine torchac (where a tiny bit of detail is allowed to be lost).

When the recipient unpacks it, they follow the rough models to put the beads back level by level, restoring the whole statue from coarse to fine (upsampling reconstruction).

And "reproduction" is: in a kitchen with an NVIDIA stove (a cloud GPU, or FAU's HPC), following this packing master's recipe, using the techniques he has practiced (the pretrained model), you pack the same batch of standard statues (8iVFB), then use a tape measure (pc_error) to measure "how much space the packing took, and how much the unpacked result differs from the original", draw the results as that fuel-consumption trade-off chart (the RD curve), and compare it against the figure in the paper — if it basically overlaps, the reproduction counts as successful.

Companions: PCGCv2 Glossary of Terms and Software.docx (concise-definition edition), PCGCv2 One-Week Reproduction Plan.md, Pre-Work Preparation.md

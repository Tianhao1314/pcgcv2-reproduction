*Translated from the author's original Chinese study notes.*

# The PCGCv2 Project · Understanding It From Scratch — A Study Handbook

> Written for you, someone with zero background (knows a little programming, has never studied AI).
> Goal: starting from the most fundamental principles of neural networks, build up layer by layer,
> until you can understand the PCGCv2 project you reproduced, and the paper by your advisor Xiumei Li.
> Compiled: 2026-05-21

---

## How to use this document

This document is **step-by-step**: Chapter 1 is the foundation, and each later chapter stands on the one before it.
Do not skip around. On the first read, just aim to "get the intuition" — all the math can be skipped.

Suggested reading approach:
- Stage 1 (getting through the conversation with the advisor): carefully read Chapters 1, 2, 3, 5, 6, 7 + the Q&A in Chapter 9.
- Stage 2 (systematically filling in the basics): together with the learning path in Chapter 10, work through the videos and tutorials, then come back and re-read.

At the end of each chapter there is a "**[Lock it in]**" line — that is the one sentence from this chapter you must take away.

---

## Chapter 0 · Where you are now, and where you need to get

Your advisor assigned you one task: reproduce a project on GitHub, **PCGCv2** (point cloud geometry compression).
You followed the steps, got it running, produced results, and wrote a report — **this part you have already completed, and done well.**

But you found that you do not know what this project "is doing". The reason is not that you are slow; it is that this project sits on the top floor of a very tall building of knowledge, and what you are missing are the few foundation floors underneath. This document fills in the foundation from the bottom up.

The whole building, from bottom to top, looks like this:

```
Floor 6   The advisor's paper (end-to-end task-aware compression)   ← the destination
Floor 5   PCGCv2 (the point cloud compression project you reproduced)
Floor 4   Learned compression (quantization / entropy coding / hyperprior)
Floor 3   Autoencoders (why neural networks can compress)
Floor 2   Training (how a neural network "learns")
Floor 1   Neural networks (what they actually are)                  ← the foundation, start here
```

---

## Chapter 1 · What exactly is a neural network

### 1.1 It is just a function

You have written functions like this:

```python
def f(x):
    return 3 * x + 2
```

Here the `3` and `2` are hard-coded by you. Give it an `x`, get a `y`.

**A neural network is also a function**, with input `x` and output `y`. Two differences:

1. Inside it there are not two numbers like `3` and `2`, but **several million numbers** (the term for them is "parameters" or "weights").
2. These several million numbers are **not written by a human** — when the network is freshly built they are all random garbage.

So a brand-new neural network is a giant function whose "parameters are all gibberish":
feed it a face photo and what it spits out is garbage. It only becomes useful through "training" (Chapter 2).

### 1.2 The smallest part: one neuron

A neural network is built from thousands upon thousands of **neurons**. What one neuron does is extremely simple:

- It receives a few input numbers, say x₁, x₂, x₃;
- It carries the same number of **weights** w₁, w₂, w₃, plus a **bias** b;
- It computes: `y = x₁·w₁ + x₂·w₂ + x₃·w₃ + b`
  (each input times its own weight, all summed, then add b);
- Finally it puts y through a very simple "bending" function (for example: anything below 0 becomes 0, anything above 0 is kept),
  this bending is called the **activation function**, and the result is this neuron's output.

**One neuron = one weighted sum + one bend. It is elementary-school arithmetic.**

The "several million parameters" the paper talks about are all the w's and b's of every neuron added together.

### 1.3 Stacking into a network: forward propagation

Arrange thousands upon thousands of neurons **in layers**: the first layer's output becomes the second layer's input,
the second layer's output becomes the third layer's input… all the way to the last layer, which spits out the final answer.

So "feed a face photo into the network and get an emotion answer", broken all the way down, is just
**several million multiplications and additions, computed layer by layer**. There is no magic, it is all arithmetic.
This process of computing from input to output is called **forward propagation (forward pass)**.

### 1.4 Images and point clouds are just a bunch of numbers inside a computer

In a computer's eyes there are no "pictures", only numbers:

- A grayscale image = a grid of numbers, each number being the brightness of that pixel (0–255).
- A color image = three such grids (red, green, blue).
- A point cloud = a big pile of (x, y, z) coordinate numbers.

So "feeding an image / a point cloud into the network" = feeding in a big handful of numbers, which can go straight into the multiply-and-add routine from 1.2.

> **[Lock it in]** A neural network = a giant function with several million knobs (parameters),
> internally all multiplications and additions; at the start the knobs are random, so it is useless.

---

## Chapter 2 · How a neural network "learns" — training

A freshly built network has all its knobs set to random values. "Training" is turning these several million knobs from random values into "useful values". Below is the most fundamental operating logic of training.

### 2.1 Loss: turning "how badly it is wrong" into a single number

Training needs a pile of **worked examples**: input + correct answer.
(The worked examples for emotion recognition = hundreds of thousands of face photos, each labeled "happy / sad / angry…")

Let the network compute an answer for one image, compare it with the correct answer, and use a formula to turn the gap into a single number —
this number is called the **loss**. A big loss = wildly wrong, a small loss = a good answer.
**The sole goal of training: make the loss as small as possible.**

The "loss function" is the scoring rule that turns the gap into a number. It is designed by a human —
**a human defines "what counts as good", and that is almost the only thing a human does in training.**

### 2.2 Gradient: which way each knob should be turned

Several million knobs — should each one be turned up or down?

For each parameter you can compute a number that tells you: **if you turn this parameter up a tiny bit,
will the loss go up or down, and how fast.** In math this number is called the **gradient**,
and you can think of it as a "slope".

### 2.3 Gradient descent: going down the mountain in fog

Imagine you are standing on a mountain, thick fog so you cannot see the path, but you want to get down to the lowest point of the valley.
What you can do: feel the slope under your feet, take one small step in the direction of steepest descent;
at the new spot, feel again, take another small step; keep going, and you will reach the valley bottom.

Training a network is exactly the same:

- "the height of the mountain" = the loss;
- "the position you are at" = the current values of the several million parameters;
- "the slope" = each parameter's gradient.

The rule is extremely simple: for a parameter, if turning it up raises the loss → turn it down a bit;
if turning it up lowers the loss → turn it up a bit; each time move only **one small step**
(the size of the step is called the **learning rate**). Move every parameter by one small step along its own gradient simultaneously,
and you have completed one "step down the mountain". Then switch to a new batch of worked examples, recompute the loss, recompute the gradients, and move one small step again.
Repeat hundreds of thousands or millions of times, and the parameters get moved to the valley bottom where "the loss is very small", and the network has learned.

This loop of "compute loss → compute gradient → move each one a small step" is called **gradient descent**,
and it is the most fundamental engine of all neural network training.

### 2.4 Backpropagation

How are the gradients of several million parameters computed efficiently? There is a clever algorithm called **backpropagation
(backpropagation)**: it starts from the loss and works the "accounting" backward layer by layer,
computing the gradients of all parameters in one pass.

For now you only need to remember: backpropagation = the method that efficiently computes, in one go, "which way every knob should be turned".
Having heard the name and knowing what it does is enough.

### 2.5 Lock in this layer

> **[Lock it in]**
> Loss = a single number measuring how badly the answer is wrong;
> training = using the "gradient descent" loop of going down the mountain in fog,
> to move several million parameters, one small step at a time, to where the loss is lowest.
> A human is only responsible for designing the "loss function" (defining what counts as good); how to turn the knobs is left to the machine to do automatically.

**This is the foundation. Everything that follows is just "swapping in a different loss function" or "swapping in a different network structure";
the underlying engine is always this chapter.** Whenever you meet a new term, come back and ask: is it changing the loss, or changing the structure?

---

## Chapter 3 · Autoencoders — why neural networks can compress

### 3.1 A network with a special structure

An **autoencoder** is a neural network with a particular structure, shaped like this:

```
input image ──▶ encoder ──▶ [bottleneck] ──▶ decoder ──▶ output image
              (compress)     very narrow    (reconstruct)
```

- **Encoder**: compresses the input image into that very narrow "bottleneck" in the middle.
- **Bottleneck**: a small clump of numbers, far smaller than the original image. The term for it is the **latent representation (latent)**.
- **Decoder**: from that small clump of numbers at the bottleneck, reconstructs the original image as best it can.

### 3.2 Why it automatically learns to compress

When training an autoencoder, the loss function requires: **the output image should be as close as possible to the input image**.
But the bottleneck in the middle is narrow and cannot hold all of the original image's information.

So the network is "forced" to do one thing: in the encoder, learn to **keep only the most important information**,
discard the less important, and compress the original image into that small bottleneck; the decoder then learns to guess the original image back from that bit of compressed information. After training:

- Encoder = a compressor (image → a small clump of numbers);
- Decoder = a decompressor (a small clump of numbers → image).

**The compression ability is "forced out by the loss function + the narrow bottleneck"; no one wrote the compression rules by hand.**
This is exactly the sentence from Chapter 2: swap in a different structure (add a bottleneck) + swap in a different loss (output ≈ input),
and you get a brand-new ability.

> **[Lock it in]** An autoencoder = encoder + narrow bottleneck + decoder;
> because the bottleneck is too narrow to hold all the information, the network is forced to learn to "grab the key points", and so it naturally becomes a compressor.

---

## Chapter 4 · The complete pipeline of learned compression

The autoencoder's bottleneck is "a small clump of numbers", but it is not yet a file you can actually store.
To become a file, two more steps are needed: quantization and entropy coding.

### 4.1 Quantization

The numbers in the bottleneck have decimals (such as 23.7194). **Quantization** is rounding them "to integers, making them coarser"
(23.7194 → 24). The benefit: numbers become simpler, easier to store. The cost: a bit of precision is lost.

**The "loss" in lossy compression mainly happens at this step.** The more aggressive the compression = the coarser the quantization = the more space saved,
but the less it resembles the original image.

### 4.2 Entropy coding (the one you asked about before)

After quantization there is a string of integers, and **entropy coding** turns it into an actual stream of 0/1 bits.
The core rule: **give frequently-occurring values short codes, and rare values long codes**, so the total number of bits is minimized.
This step itself is lossless. A common form of entropy coding is arithmetic coding (what the `torchac` library in the project does is exactly this).

### 4.3 Hyperprior

For entropy coding to compress aggressively, it must first know "the probability with which each value occurs".
The **hyperprior** is an extra small network dedicated to predicting these probabilities. The more accurate the prediction, the more aggressively entropy coding can compress.

### 4.4 The complete pipeline

```
image ─▶ encoder ─▶ latent representation ─▶ quantization ─▶ entropy coding ─▶ bitstream (file)
                                                ▲
                                       hyperprior network (predicts probabilities)
bitstream ─▶ entropy decoding ─▶ decoder ─▶ reconstructed image
```

**Bitrate (bpp)** = the size of the bitstream ÷ the number of pixels (or points), i.e. how many bits are spent on average per pixel/point.
The smaller, the more economical. This is the core metric for measuring "compression efficiency".

> **[Lock it in]** Learned compression = autoencoder (grab the key points) + quantization (make it coarse, lossy)
> + entropy coding (short codes for common values) + hyperprior (predicts probabilities to help entropy coding).

---

## Chapter 5 · The project you reproduced: what PCGCv2 is doing

**PCGCv2** = Multiscale Point Cloud Geometry Compression (DCC 2021, Nanjing University).
Repository: https://github.com/NJUVISION/PCGCv2

### 5.1 What it compresses

**Point cloud**: scan an object with a 3D scanner, and what you get is not a photo but several million little points in space,
each point having (x, y, z) coordinates; put together they form the three-dimensional shape of the object.
Point cloud data is enormous and needs to be compressed. PCGCv2 compresses only the **geometry** (the positions of the points), not the color.

### 5.2 How it does it

It is exactly the setup from Chapters 3 and 4, just with the "image version" swapped for the "point cloud version":

- It builds an autoencoder with **sparse convolution** (the MinkowskiEngine library). Sparse convolution = computing only "where there are points",
  not wasting effort computing empty air, so it is fast and saves VRAM. **This is also why you must use an NVIDIA graphics card (CUDA);
  your Mac cannot run it.**
- **Multiscale**: downsample the point cloud level by level into coarser and coarser versions, then reconstruct from coarse to fine.
- The clump of information at the bottleneck is compressed in two parts: the occupancy information goes through an octree (tmc3) for lossless compression;
  the feature attributes go through a "learned probability model + arithmetic coding (torchac)" for lossy compression.

### 5.3 What you "reproduced"

You did not invent the algorithm. What you did was: take the model the authors had trained + the standard test data, run it yourself,
and see whether you can get the same numbers as the paper. Your deviation was only 0.0025 dB — which means
"what the authors said is true, and you did not get your environment and workflow wrong". This is a sound reproduction.

### 5.4 The metrics in your report

- **bpp**: bits per point; the smaller, the more aggressive the compression.
- **D1 PSNR**: point-to-point distortion; the larger (in dB), the better the reconstruction quality.
- **D2 PSNR**: point-to-plane distortion; the larger, the better.
- **RD curve** (rate-distortion curve): bpp on the x-axis, PSNR on the y-axis. The seven models r1–r7 = seven bitrate levels,
  each producing one point, and connecting them gives the curve. The closer the curve is to the upper-left, the better.

> **[Lock it in]** PCGCv2 = applying the "learned compression" setup to point cloud geometry,
> using a sparse convolutional autoencoder, optimizing two things: compressing small (bitrate) + reconstructing accurately (distortion).

---

## Chapter 6 · The advisor's paper

Paper: **Optimized Learned Image Compression for Facial Expression Recognition**, ICIP 2025.
Author: Xiumei Li (the advisor, first author) et al. arXiv:2509.17262.

### 6.1 The problem it solves

Face images need to be compressed. But ordinary compression (including ordinary learned compression) only optimizes "how good it looks to the human eye";
compress it too aggressively and it discards the subtle features that determine the expression → the downstream AI expression recognition then becomes inaccurate.

### 6.2 Its approach: end-to-end joint optimization

It joins the "compression model" and the "expression recognition model" into one chain and trains them together. The key is that it designs a
**joint loss function** — a weighted sum of three scoring rules:

```
total loss = α·rate loss + β·MSE distortion loss + γ·cross-entropy loss
             (compress small)  (image not distorted)  (expression recognized accurately)
```

- **Rate loss**: measures how big the compressed result is (mathematically, the entropy of the latent representation).
- **MSE distortion loss**: measures the per-pixel difference between the reconstructed image and the original.
- **Cross-entropy loss**: measures how accurately the expression classification is guessed.

By tuning the three weights α, β, γ, you can find a balance among "compress small / no distortion / accurate recognition".

Recall the sentence from Chapter 2: what this paper does is essentially **designing a new loss function**
(three terms combined), and everything else is an extension of that.

### 6.3 Results

It uses the AffectNet facial expression dataset (8 emotion classes). When the two models are jointly optimized together:
expression recognition accuracy +4.04%, compression rate saving 89.12% (BD-Rate);
at low bitrates, the compressed image is even recognized slightly more accurately than the original.

> **[Lock it in]** The advisor's paper = making compression not only please the human eye, but also be responsible for both "file size" and
> "the downstream AI task" at the same time; the method is to twist three goals into one joint loss and train them end-to-end together.

---

## Chapter 7 · How the two connect into your research direction

Put Chapters 5 and 6 side by side:

| | PCGCv2 (what you reproduced) | The advisor's paper |
|---|---|---|
| What it compresses | Point cloud geometry | Face images |
| Loss function | Rate + distortion (2 terms) | Rate + distortion + **task** (3 terms) |
| New idea | —— | Downstream-task-oriented + end-to-end joint optimization |

Your advisor having you **reproduce PCGCv2 first, then read her paper** is a deliberate arrangement:

- PCGCv2 gives you the groundwork in **point cloud compression**;
- Her paper gives you the idea of **"task-aware + end-to-end joint optimization"**.

Now look again at the research direction on her homepage: "**scalable** learned point cloud compression, supporting decoding
multiple quality levels from a single bitstream, oriented toward **human-machine collaborative** use"; the master's topic she supervises is "scalable point cloud compression for human-machine collaboration".

**Putting it together: she most likely wants you to combine these two — to do human-machine-collaboration-oriented, task-aware point cloud compression.**
Your upcoming research topic / internship direction is basically here.

---

## Chapter 8 · A quick-reference mini glossary

| Term | One-sentence explanation |
|---|---|
| Neural network | A giant function with several million knobs (parameters), internally all multiply-and-add |
| Parameter / weight | The several million tunable numbers inside the network |
| Forward propagation | The process of computing the input layer-by-layer with multiply-and-add to the output |
| Loss | A single number measuring how badly the answer is wrong |
| Loss function | The scoring rule that turns the "gap" into a loss; designed by a human |
| Gradient | For a given parameter, "will turning it up raise or lower the loss, and how fast" — the slope |
| Gradient descent | The loop that uses the gradient to move parameters one small step at a time to where the loss is lowest |
| Backpropagation | The algorithm that efficiently computes all parameters' gradients in one go |
| Training / optimization | Running gradient descent to tune the network from "random" to "useful" |
| Fine-tuning | Not training from scratch, but taking an already-trained model and adjusting it slightly |
| Autoencoder | Encoder + narrow bottleneck + decoder, forced to learn to compress |
| Latent representation (latent) | The small clump of compressed numbers squeezed out by the encoder |
| Quantization | Rounding numbers to make them coarser; the "loss" of lossy compression is at this step |
| Entropy coding | Turning numbers into a bitstream, with short codes for common values |
| Hyperprior | A small network that predicts probabilities to help entropy coding compress more aggressively |
| End-to-end | Connecting from input to final result as one chain, training the whole chain together |
| Joint optimization | Tuning the parameters of multiple models simultaneously so they adapt to each other |
| bpp | Bits per pixel/per point; the smaller, the more aggressive the compression |
| PSNR (D1/D2) | Reconstruction quality; the larger (in dB), the better |
| RD curve | bpp on the x-axis, PSNR on the y-axis; the closer to the upper-left, the better |
| BD-Rate | How much bitrate is saved on average at equal quality; the more negative, the better |
| Sparse convolution | Convolution that computes only "where there are points", dedicated to point clouds, requires CUDA |
| FER | Facial Expression Recognition |

---

## Chapter 9 · Questions the advisor might ask + reference answers

Answer along these lines, do not memorize them rote; saying it in your own words is best.

**Q: What does this project (PCGCv2) do?**
A: It is a method that uses deep learning to compress point cloud geometry. A point cloud is a massive set of spatial points obtained from 3D scanning,
the data volume is large so it needs to be compressed. It uses a sparse convolutional autoencoder to compress the point cloud into a very small representation and reconstruct it.
What I reproduced is its rate-distortion curve on the 8iVFB dataset, and it matched the authors' official results.

**Q: What exactly did the "reproduction" you mention reproduce?**
A: Not reinventing the algorithm, but using the authors' pretrained models and standard test data to produce
bpp and D1/D2 PSNR myself, draw the RD curve, and compare it against the official results. My deviation was only 0.0025 dB,
which means the environment and workflow are both correct.

**Q: What is entropy coding? What role does it play here?**
A: Entropy coding is the foundation of lossless compression; the idea is to give common values short codes and rare values long codes.
The neural network is responsible for predicting probabilities, and entropy coding is responsible for converting probabilities into an actual bitstream.
It is the step where compression is realized; the bpp metric is its product.

**Q: Why do you need a GPU; why won't a Mac do?**
A: It depends on the sparse convolution library MinkowskiEngine, which strictly requires NVIDIA's CUDA.
A Mac has no CUDA, so I did the entire reproduction on a cloud GPU (RTX 3090).

**Q: You read my paper — talk through its core idea.**
A: Your paper is about image compression oriented toward facial expression recognition. Ordinary compression only optimizes for the human eye;
compress it too aggressively and it discards expression features, so recognition gets worse. Your approach is to join the compression model and the recognition model end-to-end,
train them together with a joint loss (rate + MSE + cross-entropy), so that compression actively preserves the features recognition needs.
Under joint optimization, accuracy improved by 4% and bitrate was saved by 89%.

**Q: What is the relationship between PCGCv2 and my paper?**
A: Both are learned compression. PCGCv2's loss only concerns the two terms of rate and distortion; your paper adds a third term,
the "task loss". I understand that having me reproduce PCGCv2 first lays the groundwork in point cloud compression, and reading your paper is
to learn the idea of "task-aware + end-to-end joint optimization", and next it might be combining the two —
doing task-aware, human-machine-collaboration-oriented point cloud compression.

**Q: What was the biggest difficulty in the reproduction?**
A: Building MinkowskiEngine from source. It is extremely sensitive to versions; I strictly used the
Python 3.8 / CUDA 11.1 / PyTorch 1.8.1 combination and only then did it compile successfully on the first try.
In addition, the overseas network environment also had some pitfalls (switching to RunPod, using aria2 for multi-threaded downloads, etc.), all recorded in the report.

---

## Chapter 10 · An advanced learning path (Stage 2)

No need to rush; go in order, each step builds on the previous one.

1. **Neural network intuition**: watch 3Blue1Brown's "neural networks" series (4 short videos,
   with Chinese subtitles, purely about intuition, no piling up of formulas). Solidify the feel of Chapters 1 and 2 of this document.

2. **Do one yourself**: you know Python, so spend a day or two going through PyTorch's official getting-started tutorial
   ("60 Minute Blitz") and train a small network yourself. Concepts will go from "heard of" to "done".

3. **A systematic course (optional)**: if you want a more solid grounding, take an introductory deep learning course (such as Andrew Ng's course).

4. **Image compression background**: get to know the "learned image compression" framework of Ballé 2017/2018 —
   your advisor's paper is built on this line of work.

5. **Point cloud basics**: fill in a bit on point clouds, octrees, and PSNR.

6. **Re-read**: re-read PCGCv2's README and the advisor's paper. You will find you can understand them.

---

## Appendix: update log for this handbook

| Date | Update content |
|---|---|
| 2026-05-21 | First version: Chapters 1–10, built from the fundamentals of neural networks up to the advisor's paper |

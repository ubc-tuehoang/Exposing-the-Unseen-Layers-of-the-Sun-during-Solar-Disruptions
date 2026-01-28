# Unsupervised ML in This Project 

This document explains, in everyday words, how the solar slicing code uses **unsupervised machine learning** to turn solar images into separate "layer" images—and what that means.

---

## What does "unsupervised" mean?

**Unsupervised** means the computer is not told what to look for. No one labels "this is a sunspot" or "this is a flare." The program only gets the raw pixel values and tries to find **patterns** or **groups** on its own.

In this project we use five methods:

- **PCA, NMF, and ICA** — each splits the image into a few "layers" that, when combined, resemble the original.
- **Robust PCA (RPCA)** — splits the data into **background** (smooth, low-rank) and **anomalies** (sparse, unusual pixels). Used in stack mode only.
- **K-means** — groups pixels by how bright they are and where they are; each group becomes its own layer.

PCA, NMF, ICA, and K-means use `scikit-learn`. RPCA is implemented in the script (IALM algorithm, pure numpy).

---

## How the program sees the data

### When you give it several images (stack mode)

If you pass in multiple solar images (e.g. three different times):

- For every pixel, the program looks at **one number from each image** (how bright that spot is in image 1, image 2, image 3).
- It then tries to explain those triplets using a few "layers." Each layer is a full image: one picture per pattern it found.
- So you get a small set of images (e.g. 3) that summarize the main ways the Sun picture changes across the three inputs.

### When you give it one image (single-image mode)

If you pass in only one image:

- For every pixel it uses three numbers: **how bright** it is, and **where** it is (horizontal and vertical position, scaled to 0–1).
- The program then splits that into layers (PCA, NMF, ICA) or **clusters** pixels into regions (K-means). Each layer or cluster is turned into its own image.

---

## PCA (Principal Component Analysis)

- **In plain language:** PCA looks for the main directions in which the numbers "spread out" the most. The first layer is the single pattern that explains the most variation; the next layer is the next-biggest pattern that is *different* from the first, and so on.
- **What you get:** One image per pattern (e.g. `stack_pca_1.png`, `stack_pca_2.png`). Brighter and darker areas in each image show where that pattern is strong or weak.
- **Why it’s useful here:** With several images, PCA separates the main ways the Sun view changes over time or across images. With one image (using brightness + position), it can highlight different regions or behaviors without you having to define them.

---

## NMF (Non-negative Matrix Factorization)

- **In plain language:** NMF assumes the data is built by **adding** a few positive "parts" together. So everything is positive (no negative brightness). It looks for a small set of parts that, when combined, closely match the original.
- **What you get:** One image per "part" (e.g. `stack_nmf_1.png`, `stack_nmf_2.png`). Each part is a layer; together they add up to something like the input.
- **Why it’s useful here:** Solar images are naturally non-negative (brightness ≥ 0). NMF often finds parts that match physical intuition—e.g. one part for quiet Sun, another for active or bright regions—without anyone labeling them.

---

## ICA (Independent Component Analysis)

- **In plain language:** ICA assumes the data is a **mix** of a few hidden "sources," and it tries to recover those sources by making them as **independent** as possible—meaning knowing one source doesn’t help you predict another.
- **What you get:** One image per recovered source (e.g. `stack_ica_1.png`, `stack_ica_2.png`). Each image is one "unmixed" layer.
- **Why it’s useful here:** ICA can pull apart mixed contributions—e.g. different physical processes or time behaviors—when they act somewhat independently. The layers often look different from PCA or NMF, so it gives you another view of the same data.

---

## Robust PCA (RPCA) — background vs. anomalies

- **In plain language:** RPCA assumes the data is **background plus anomalies**. It looks for a **smooth, low-rank** part (the background) and a **sparse** part (a few pixels that stand out). No one marks "this is an anomaly"—the method finds them from the numbers.
- **What you get:** Two images: `stack_rpca_background.png` (the smooth background) and `stack_rpca_anomalies.png` (where the anomalies are; brighter = stronger anomaly).
- **Why it’s useful here:** In stack mode, RPCA can separate "quiet Sun" or slowly varying structure (background) from flares, bright spots, or outliers (anomalies). It’s the method that explicitly gives **background vs. anomalies**.

---

## K-means (clustering)

- **In plain language:** K-means splits pixels into **K groups** (e.g. 4) by **similarity**: it puts pixels that are similar in brightness and position into the same group. No one defines the groups in advance; the program finds them from the data.
- **What you get:** One image per group (e.g. `kmeans_1.png`, `kmeans_2.png`). In each image, only the pixels in that group keep their brightness; the rest are dark. So you see "which region belongs to which cluster."
- **Why it’s useful here:** Used only in single-image mode. It can separate things like "bright center vs. dimmer limb" or "very bright spots vs. background" just from brightness and position, without labels.

---

## Short summary

| Method    | In one sentence |
|-----------|------------------|
| **PCA**   | Finds the main patterns in how the numbers vary and turns each pattern into its own layer image. |
| **NMF**   | Finds a few positive "parts" that add up to the data; each part becomes a layer image. |
| **ICA**   | Tries to unmix a few independent "sources" from the data; each source becomes a layer image. |
| **Robust PCA** | Splits the data into a smooth **background** and **anomaly** layers (stack mode only). |
| **K-means** | Groups pixels by brightness and position; each group is shown as its own layer image. |

All five are **unsupervised**: they use only pixel values (and position when in single-image mode). No manual labels or physics equations are required. The script turns their results into grayscale images so you can look at and compare the layers.


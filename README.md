# Utilization of 360° Media for Distortion-Aware 3D Scence Reconstruction using Gaussian-Based Methods with ML Depth Priors 

**M.Sc. Thesis - Computer Simulation in Science**
Bergische Universität Wuppertal | TMDT AR/VR Lab
Abhilash Ashokrao Masane

---

## What This Is

Most 3D Gaussian Splatting pipelines assume a pinhole camera - a flat image with a single, narrow field of view. But the cameras increasingly used in robotics, AR/VR capture, and indoor scanning are **fisheye and 360° panoramic cameras**, which see the whole room (or whole sphere) in a single shot. Feeding that kind of image into a pinhole-based pipeline either throws away most of the field of view or distorts the reconstruction badly.

This thesis builds and evaluates a reconstruction pipeline, on top of NVIDIA's **3D Gaussian Unscented Transform (3DGUT)**, that works directly with fisheye and equirectangular (360°) images - no undistortion step, no stitching artifacts forced into the geometry - and asks a simple question:

> **Does accounting for the non-uniform way a 360° camera samples the world actually improve the 3D reconstruction, or is "just feed it raw" already good enough?**

## The Core Idea

A 360° image doesn't sample the world evenly. In an equirectangular panorama, pixels near the poles (top/bottom) represent a tiny sliver of the actual scene, while pixels near the equator represent much more. The same is true, in a different geometric form, for a fisheye lens — the center and the edge of the image don't carry equal information per pixel.

Standard 3D Gaussian Splatting training treats every pixel equally. This thesis adds a **solid-angle-aware training loss** that weights each pixel by how much of the scene it actually represents, so the model spends its capacity where the information is — not where the projection happens to put more pixels.

## What Was Built

- **Native ERP (equirectangular) camera support in 3DGUT** : the framework only handled pinhole and fisheye out of the box; this adds full equirectangular ray generation and registers it as a usable COLMAP camera model.
- **A generalized, camera-agnostic solid-angle loss** : automatically detects whether a training batch is fisheye or ERP and applies the correct weighting formula, with zero hardcoded assumptions about which dataset is being trained.
- **A latitude-stratified evaluation framework** : splits any 360° reconstruction into polar / mid-latitude / equatorial bands and reports quality per band, because a single global PSNR number hides exactly the effect this thesis is trying to measure.
- **A controlled, same-scene ablation** comparing pinhole, fisheye-direct, and distortion-aware fisheye/ERP reconstruction on identical physical scenes.
- **Depth-guided Gaussian initialization** (in progress) using both ground-truth depth and ML-estimated depth (Depth Anything V2 via cubemap projection) for scenes without depth sensors.

## Headline Result

Direct fisheye reconstruction — no undistortion preprocessing — outperforms the standard pinhole baseline by **+2.3 to +2.7 dB PSNR** on the same physical indoor scene. This validates the core premise: forcing 360° media through a pinhole pipeline costs reconstruction quality, and 3DGUT's Unscented Transform handles the native camera geometry correctly without needing a custom CUDA kernel per camera model.

The solid-angle weighting shows a consistent improvement during early training across both ERP and fisheye data, and a latitude-stratified breakdown confirms the mechanism works exactly as designed — redistributing reconstruction quality from oversampled polar regions toward the more informative mid-latitude bands — even in cases where the net effect on global PSNR is small.

## Methodology, In Short

| Variant | What it tests |
|---|---|
| **V1 - Pinhole baseline** | Standard 3DGS/3DGUT on undistorted perspective images. The reference point. |
| **V2 - 360° direct** | Same scene, trained directly on raw fisheye or equirectangular images, uniform loss. |
| **V3 - Distortion-aware** | Same as V2, but with the solid-angle-weighted loss - pixels near the poles (ERP) or image edge (fisheye) are weighted by how much actual scene area they represent. |
| **V4 - Depth-guided init** | Same as V3, but Gaussians are seeded from a dense depth map (ground truth or ML-estimated) instead of sparse COLMAP points. |

Each variant is evaluated with PSNR, SSIM, and LPIPS, plus a custom latitude-stratified breakdown for 360° data, across both real captured scenes (FIORD indoor fisheye) and synthetic data (OB3D equirectangular) to separate genuine geometric effects from dataset-specific noise.

## Status

Actively in progress. V1–V3 are complete with canonical results across multiple scenes; V4 (depth-guided initialization) is the current focus, followed by full-resolution validation and thesis writing.

---

*Built on [NVIDIA 3DGUT](https://github.com/nv-tlabs/3dgrut). This repository accompanies an M.Sc. thesis at Bergische Universität Wuppertal, expected submission September 2026.*

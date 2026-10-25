---
layout: page
title: Dronalize
description: An open-source toolbox for unifying trajectory prediction research across bird's-eye-view and motion-forecasting datasets.
img: assets/img/dronalize/card.png
importance: 1
category: research
related_publications: false
toc:
  sidebar: left
---

{% include figure.liquid loading="eager" path="assets/img/dronalize/logo.png" class="img-fluid rounded z-depth-1" %}

**Dronalize** is an open-source toolbox that aims to consolidate the research and development
of trajectory prediction models. It started as a smaller project focused on drone-captured,
bird's-eye-view datasets, and it has since grown to support a wide range of popular motion
forecasting benchmarks. The toolbox provides utilities for data preprocessing, visualization,
and evaluation, together with a model-development pipeline tailored to machine-learning-based
trajectory prediction.

It is built on [PyTorch](https://pytorch.org/), [PyTorch Geometric](https://pytorch-geometric.readthedocs.io/),
and [PyTorch Lightning](https://lightning.ai/docs/pytorch/stable/), and it is released under
the Apache 2.0 license.

<div class="text-center mt-2 mb-4">
  <a class="btn btn-outline-primary mb-2" href="https://github.com/westny/dronalize" target="_blank" rel="noopener noreferrer">Code (GitHub)</a>
  <a class="btn btn-outline-primary mb-2" href="https://arxiv.org/abs/2405.00604" target="_blank" rel="noopener noreferrer">Paper (arXiv)</a>
  <a class="btn btn-outline-primary mb-2" href="https://hub.docker.com/r/westny/dronalize/tags" target="_blank" rel="noopener noreferrer">Docker Hub</a>
</div>

## Motivation

Trajectory prediction research is spread across many datasets, each with its own formats,
coordinate conventions, map representations, and evaluation scripts. That fragmentation makes
it hard to compare methods fairly or to reuse code across benchmarks. The accompanying paper,
*Toward Unified Practices in Trajectory Prediction Research on Bird's-Eye-View Datasets*,
argues for a common set of practices, and Dronalize is the practical tooling that puts them
into effect. The goal is a single, consistent pipeline from raw dataset to trained model to
evaluation, so that researchers can spend their time on modeling rather than on plumbing.

## Supported datasets

Dronalize began with drone-captured datasets and now covers more than twenty benchmarks that
span highways, intersections, roundabouts, pedestrian scenes, and dense urban traffic. Most
are freely available for non-commercial use, though some require registration.

- **Drone-captured:** highD, inD, rounD, exiD, uniD, openDD, AD4CHE, SIND, A43, INTERACTION
- **Instrumented-vehicle and other:** Argoverse 1 and 2, Waymo Open Motion Dataset, nuScenes, Lyft Level 5, View-of-Delft, ApolloScape, NGSIM (I-80 and US-101), ETH, UCY

Each dataset is handled through a dedicated preprocessing pipeline and configuration, which
converts its native format into a common representation (agent trajectories plus a lane graph)
that the rest of the toolbox consumes. The full list, with download links and per-dataset
instructions, lives on the [datasets page](https://github.com/westny/dronalize/blob/main/docs/datasets.md)
of the repository.

## A unified pipeline

The toolbox is organized as a sequence of interchangeable stages, so that a model written
against one dataset runs against any other with only a change of configuration:

1. **Preprocessing** converts each dataset into the common trajectory-plus-lane-graph format.
2. **Data loading** serves the processed scenes as batched graphs through PyTorch Geometric.
3. **Modeling** provides a base interface and reference models to build on.
4. **Training** is driven by PyTorch Lightning, with reproducible configs and containers.
5. **Metrics** implements the standard prediction measures (such as ADE, FDE, and miss rate) consistently across datasets.

Because preprocessing, data loading, training, and evaluation share one representation, a
result obtained on one benchmark is directly comparable to a result on another.

## Visualization

A core feature of the toolbox is its visualization utilities, which render scenes, lane
graphs, and predicted trajectories directly from the processed data. These make it easy to
inspect a dataset, debug a preprocessing step, or present qualitative results.

<div class="row">
  <div class="col-md-6 mt-3 mt-md-0">
    <figure>
      <img src="{{ 'assets/img/dronalize/scene_interaction.gif' | relative_url }}" alt="Animated INTERACTION dataset scene with lane graph" class="img-fluid rounded z-depth-1" loading="lazy">
      <figcaption class="caption">An INTERACTION scene, showing the lane graph (dotted) and tracked agents (blue) over time.</figcaption>
    </figure>
  </div>
  <div class="col-md-6 mt-3 mt-md-0">
    <figure>
      <img src="{{ 'assets/img/dronalize/scene_exid.gif' | relative_url }}" alt="Animated exiD dataset highway scene with lane graph" class="img-fluid rounded z-depth-1" loading="lazy">
      <figcaption class="caption">A highway scene from the exiD dataset, with its lane graph and tracked vehicles.</figcaption>
    </figure>
  </div>
</div>

## Baseline models

Two reference models ship with the toolbox so that new methods have a point of comparison:

- A lightweight **prototype model** that serves as a simple, readable baseline.
- **MTP-GO**, a graph-based probabilistic predictor built on neural ordinary differential equations, which provides a stronger reference point.

## Example use case: transfer across datasets

One motivation for a unified pipeline is that it makes cross-dataset experiments
straightforward. In the accompanying paper, the toolbox is used to study domain adaptation and
generalization: the prototype model is pre-trained on one dataset and fine-tuned on another,
and vice versa. Even with a very simple architecture, transfer learning yields clear
improvements, which is only practical to measure because the datasets share one representation
and one evaluation path.

{% include figure.liquid loading="lazy" path="assets/img/dronalize/transfer_learning.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Transfer learning between datasets: the prototype model is pre-trained on INTERACTION and fine-tuned on inD, and vice versa, giving consistent gains even with a simple model." %}

The toolbox also underpins our follow-up work on transferability,
[Unveiling Transferability in Trajectory Prediction via Latent Scene Embeddings](https://arxiv.org/abs/2606.30777),
which was presented at ECCV 2026. That study uses Dronalize's unified multi-dataset pipeline
to analyze how learned scene representations transfer across benchmarks. Code and a project
page are available at [westny.github.io/transferatlas](https://westny.github.io/transferatlas/).

## Related work

The toolbox grew out of, and continues to support, several research projects on behavior
prediction:

- [Unveiling Transferability in Trajectory Prediction via Latent Scene Embeddings](https://arxiv.org/abs/2606.30777), *ECCV*, 2026.
- [Diffusion-Based Environment-Aware Trajectory Prediction](https://arxiv.org/abs/2403.11643), arXiv preprint, 2024.
- [MTP-GO: Graph-Based Probabilistic Multi-Agent Trajectory Prediction with Neural ODEs](https://arxiv.org/abs/2302.00735), *IEEE Transactions on Intelligent Vehicles*, 2023.
- [Evaluation of Differentially Constrained Motion Models for Graph-Based Trajectory Prediction](https://arxiv.org/abs/2304.05116), *IEEE Intelligent Vehicles Symposium (IV)*, 2023.

## Cite

If you use the toolbox in your research, please consider citing the paper:

```bibtex
@inproceedings{westny2025dronalize,
  title={Toward Unified Practices in Trajectory Prediction Research on Bird's-Eye-View Datasets},
  author={Westny, Theodor and Olofsson, Bj{\"o}rn and Frisk, Erik},
  booktitle={IEEE Intelligent Vehicles Symposium (IV)},
  year={2025}
}
```

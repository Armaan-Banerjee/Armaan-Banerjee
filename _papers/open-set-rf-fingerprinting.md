---
title: "C²T-OpenMax: A Novel Open-Set WiFi RF Fingerprinting Method via Center Constrained Learning and Confidence-Guided Tail Modeling"
date: 2026-10-23
authors: "Yuanyu Zhang, Junjie Yang, Ji He, Shuangrui Zhao, Lele Zheng, Yulong Shen"
paper_url: "https://arxiv.org/pdf/2609.02007"
---

Radio frequency fingerprinting authenticates devices from using the imperfections in the hardware of a transmitter , the paper they use a neural network to help with this

In 802.11 (wifi), Channel State Information (CSI) provides amplitude and phase responses, which models can use to train for device repreentations

current RFF assumes the set of devices remains the same, no new unathentiated devices

THey use DeepCRF with OpenMax, DeepCRF uses channel simulatin and guassian noise (noise sampled from normal distribution) to learn device rerpesentations in dfiferent channel conditions

OpenMax uses the activation vectores of correctly calssified samples to calcualte a mean activiation vector for each known class, it then finds the distance from each activation vector to the mean activation vector, and selects samples to fit a Weibull distribution

Weibull distrbituion: has 2 parameters lambda and k (example from wikipedia)

![Weibull probability density curves with lambda = 1 and k = 0.5, 1.0, 1.5 and 5.0]({{ '/assets/papers/open-set-rf-fingerprinting/figure-1.webp' | relative_url }})

The paper adds Confidence guided tail modeling, which selects only correctly classified high confidence training samples to get better mean activations

The confidence comes from sfotmax on the logits, and removing those below a threshold and then building the Weibull

They tested this in line of sight and non line of sight networks, static and dynamic and indoor and outdoor.

The C^2T-OpenMax algorithm outperfmrs other RFF like OpenMax and CSSR by roughly 5 percentage points in most scenarios

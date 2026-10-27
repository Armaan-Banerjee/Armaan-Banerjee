---
title: "Secrets in Preamble: Practical and Robust PHY Information Hiding for Wi-Fi"
date: 2026-10-23
authors: "Rui Pan, Guanxiong Shen, Linning Peng, Xinyu Qi, Junqing Zhang"
paper_url: "https://arxiv.org/pdf/2610.13367"
---

## Abstract

- Physical layer (PHY) information hiding concels secrets withing the PHY waveforms without compromisng the communication, allowing covert communication, PHY steganography and watermarking
- Wi-FI historically suffers from restricted steganographic capacity and not being very robust against the environment
- The paper proposes using the 902.11 legacy short training field (L-STF)
- The secret embedding and extraction processes use a pair of neural entowrks which learn optimal embedding rules and maximise steganographic capacity

## Introduction

- External observers only receive and demodulate the conventional payload data without seeing the concealed secret, wheras authorised receivers can recover the embedded information, in PHY embedding
- Wi-Fi becoming the default for WLAN makes it a significant research interest
- Previous studies have suggested that the receiver can extract secrets throught he channel state information, but only works in static environments and was very limited in capacity
- Another approach is to embed secrets in the IQ constellation diagram, such as a dirty constellation sccheme to hide secrets, or a proposed pseudo-noise asymettric shift keying (PN-ASK) where information is carried in the amplitude however these degrade the performance of the Wi-Fi link
- Hiding secrets faces 2 limitation, it must not interefere with standard demodulation process, and as Wi-Fi has a wide bandwith, the secrets must be robust enough to survive propogation effects like multi-path fading and noise
- The neural networks proposed to embed the secrets use an encoder-decoder architecture and do joint trainsing
- A TGn channel simulator is incorportated during training to improve the ability for the neural networks to be deployed in more applications

## System overview

- Alice embeds a secret message into the 802.11 packet using the pre-trained neural netowrk, which introduce subtle distortions tot he waveform, at the receiver, BOB uses the paired neural network to recover the secrets
- The embedder also fills int he legacy long training field (L-LTF) for channel estimation, legacy signal field (L-SIG) for transmission paramers like rate and payload length, which are modulated into the DATA field
- BOB does standard Wi-FI demodulation and obtains the channel state information from L-LTF, this is used to equalise teh L-STF region to help against channel induced distortions

## Secret Embedding

- The distortions to a single STS is repeated 10 times to contruct the L-STF
- THe L-STF should contain 10 identical STS and allows for packet detection and carrier frequency offset compensation
- THe L-LTF follows the L-STF, and includes a guard interval and 2 identical long term training symbols, the guard interval being the latter half of an LTS
- The legacy signal (L-SIG) conveys modulation rate, framel legnth, parity check information and tail bits, and teh DATA field carries PHY service data unit
- This is why L-STF was picked, using L-LTF would lease to errors in channel state information, imparing demodulation of the DATA field, L-SIG and DATA would also compromise integrity
- L-STF occupies 12/64 subcarriers for wifi 1,2,3. This was formed using a 64 point inverse fast fourier transform to generate 4 16-sample time domain STS, the L-STF comprises of 10 identical STS each comprising 16 samples
- L-STF is used for packet detection as the receiver detecs WI-Fi signals which mach the periodic characteristics of the L-STF, it is also used for CFO compensation where CFO is the estimate of the phase differences between adjacent periods
- By ensuring the embedded secrets do not affects the cyclical nature of the L-STF, functionality should remain of the field
- The secret embedder and extractor are implemented as 1D convolutional neural netowrks along the 12 active subcarriers
- After the neural network has run, the distorted frequency domain STS are converted to the time domain

## Secret extraction

- At the receiver BOB executes packet detection and coarse CFO compensation using the L-STF, and uses the L-LTF for channel state estimation, this is then used to equalise the L-SIG and DATA fields
- As Wi-Fi has high bandwift, multi-ath induced fading often affects the subcarriers, the channel state estimation from the L-LTF is used to also equalise the L-STF, then uses the FFT to convert to frequence domain and then runs it thorugh the paired neural network

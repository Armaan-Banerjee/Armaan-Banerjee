---
title: "Do Electromagnetic Side-Channel Attacks Threaten Electronic Polling Stations? Scenarios and Recommendations"
date: 2026-10-23
authors: "Lucas Brito, Leonardo Teodoro, Pedro Tomaz, Alyson Isaluski, Leandro Hyeda, Antonio Oliveira-Jr, Saulo Queiroz"
paper_url: "https://arxiv.org/pdf/2609.28209"
---

## Abstract

- Investigates wether there is a threat to ballot secrecy in the Brazilian electronic voting machine through side channel attacks, known as TEMPEST attacks
- Screen content can be reconstructed by interceptng em radiation associated with the target
- The experiments shows that using SDR to record EM radiation and use it as a side channel attack depends on the lack of oversight

## Introduction

- Recently, the city of Para removed a city councilor from office for recruiting voters to wear glasses with embeded minature cameras to violate ballot secrecy
- In a TEMPEST attack, an attacker uses SDR to intercept the em radiation from the transmission of video signal through the system's components
- The radiation is then processed and a successful attack can reconstruct the video image carried by the signal

## Video signal

- The VGA signal used by at least 2 types of the ballot box (UEB), uses pulse amplitude modification, where information is carried in the amplitude of the rectangualr pulses, the time interval of a pixel is inversely proportional to the number of pixels on screen and the frame refresh rate
- The fourier transform of the VGA pulse is the time period times a sinc function (fourier of rectangle is always sinc function)
- The fourier transform therefore shows the bandwith used by the VGA cables, although the sinc function has 0s at multiples of the refresh rate, in reality, badwith limitations and other inefficiencies means that the TEMPEST signal exhits observanble energy at those frequencies, meaning an adversary can use any nearby frequency to process

## Attack scenarios

- Scenario 1: An adversary with access to a place next to the polling station room installs an SDR computer setup, the attack exploits the fact that voting booths are often positioned near walls to prevent chance of other voters seeing their choices, this means that physical proximity may allow the capture of em leakage
- Scenario 2: Someone with legitimate access to the polling station could carry a portale SDR which can restore raw signals for offline analysis during voting hours
- In Scenario 1 they placed the SDR 1 meter away from the UEB with a wall seperating them, in scenario 2, the distance between SDR and UEB was less tha 0.5m, and iin both signals were captured by a USRP B200 radio tuned to a harmonic of the pixel rate
- THe results show that the risk of violating ballot secrecy strongly depends on the proximity between adversary and polling station
  ![Figure 2 from the paper: voting machine screens reconstructed through a TEMPEST attack, at about 1 meter through a masonry wall and at under 0.5 meters with no obstacles]({{ '/assets/papers/electronic-voting-tempest/figure-1.png' | relative_url }})

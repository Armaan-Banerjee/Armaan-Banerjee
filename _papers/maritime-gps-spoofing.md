---
title: "She Spoofed Sea Ships by the Sea Shore: Measuring Large-Scale GPS Spoofing in Global Maritime Traffic"
date: 2026-10-23
authors: "Anna Raymaker, Ryan Von Brock, Ryan Pickren, Animesh Chhotaray, Frank Li, Saman Zonouz, Raheem Beyah"
paper_url: "https://arxiv.org/pdf/2609.37676"
---

## Abstract

- Uses Automatic Identification System (AIS) data to get the GPS coordinates broadcasted by ships 
- Found 31 hotpsots, with 22 showsing evidence of spatial and temporal GPS spoofing 

## Introduction

- Over 80% of marine casualties involved human or navigational factors in 2024 
- MSC Antonia was a 75,000 ton container ship grounded in the red sea due to GPS spoofing 
- GPS spoofing displaces reported positions of many vessels and so produces a singature, as the vessels report implauisble movements 

## Background

- AIS is transponder base and required for all vessles above 300 gross tons and all passenger vessels, each AIS message broadcasts the vessel's Maritime Mobile Service Identity (MMSI) along with gnss position, speed, course and voyage data
- Civilian gnss is unecrypted so vulnerbale to spoofign attack 
- Maritime industry is particularly vulnerable with attacks against AIS, satelite terminals, communication stacks and cyber-physical ship control systems 

## Methodology

- Per-vessel anomaly analysis: each MMSI is a single vessel and flags motion that seems imjplausable like abrupt jumps, implausible speeds, or position over land however may not be gps spoofing yet, as could be a faulty transponder 
- Pipeline is input AIS parsing, cleaning, kalman prediction which predicts next check, violation check of wether over land, speed > 60 knotts or deviation > 5km, then runs though a finite state machine to update gps sppofing state 
- The Kalman filter ([https://kalmanfilter.net/](https://kalmanfilter.net/)) adopts a constant velocity motion, and assumes a vessel maintains speed and heaidng between updates, as updates are failry close together, this assumption is good for short term approximation as vessesl cannto change speed or direction quickly 
- The vessel's state vector has 4 dimensions, lattitude, longitude, and 2 velocity components, the measurement vector has the AIS positions 
- The predicted positions is compared with the new AIS observation, and then use the Haversine great-circle distance between the predicted and observed positions 
- The Finite State Machine labels each minute as anomalous if any of its AIS points trigger a violation, a 30 minute rolling window is then used across time and calculate the ratio of anomolous points to total points, a minimum of 70% of points have to be anolomous before declaring gps spoofing 
- For eveyr anomolous segment they extract the last clean point before and after and are connected to form a jump line, to show the vessels displacement during the anomaly episode 
- DBSCAN is used to cluster nearby inteersections to anomaly zones 
- Vessels that never share spatial or temporal overlap with other vessels are removed, and remove spatial outliers whose anomalous locatiors are 3x intra-cluster distance from the centre 

## Results

- Found anomalies in the black sea, east mediterranean, red sea, north sea and baltic, east asia seas, inland china, inland russia, capsian sea, atlantic corridor, gulf of mexico and panama canal, strait of hormuz 
- ![Map of numbered anomaly zones across Europe, Russia, the Middle East and East Asia]({{ '/assets/papers/maritime-gps-spoofing/figure-1.png' | relative_url }})
- ![Map of anomaly zones 10 in the Gulf of Mexico and 23 at the Panama Canal]({{ '/assets/papers/maritime-gps-spoofing/figure-2.png' | relative_url }})

## Identifying gps spoofing

- Of the 31 anomalous zones, they found which ones were gps spoofing using:
- Corroborated spoofing - a unique spoofing singature with exernal corroboration using incident report and over documentation 
- Newly identified spoofing- the signature itself is strong enough evidence 
- Suggestive - anomalies shared across vessels but with weaker signatures 

## Cast study

- The black sea had the most sustained spoofing affecting over 1,000 vehicels, circular crop-circle loops
  ![Figures 11 and 12 from the paper: spoofed AIS tracks and circular loops off Sochi in the Black Sea, and spoofed points across Israel and the Red Sea]({{ '/assets/papers/maritime-gps-spoofing/figure-3.png' | relative_url }})
- There are also abrupt, long-range jumps across the sea
- Across the red sea, point convergence and linear displacement was used, multiple vessels had the same point where multiple AIS tracks jumped to the coordinates of the Queen Alia International Airport in Jordan and near port sudan, parallel false tracks run for10s of kilometers from the coast, running the same latitude

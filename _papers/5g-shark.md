---
title: "5G-Shark: A Network Security Auditor for 5G Subscriber Privacy and Unauthenticated Signalling Resilience"
date: 2026-10-23
authors: "Oscar Lasierra, Gines Garcia-Aviles, Antonio Skarmeta, Xavier Costa-Pérez"
paper_url: "https://arxiv.org/pdf/2609.24656"
---

## Abstract

- 5G was meant to close the privacy gaps of earlier generations by concealing the subscriber's permanent identity, resisting downgrades to older generations and protecting against location tracking
- The paper presents 5G-Shark, a low cost auditing tool built from open-source software and an SDR, which abuses the standard cell reselection procedure to pull a phone (User Equipment, UE) onto a fake base station without jamming or sending malformed packets
- From there it requests subscriber identifiers, forces downgrades to 4G/3G using crafted Registration Reject codes and can cause denial of service
- Each issue is classified as either a protocol-design limitation (exploitable even if the network follows the standard perfectly) or an implementation gap (caused by an operator or modem picking a weak but allowed option)
- Also found that some commercial operators hand out temporary identifiers in near-sequential steps, so users can be tracked even when their permanent identity is properly concealed

## Introduction

- In 4G and earlier, the permanent identifier (IMSI, International Mobile Subscriber Identity) was sent over the air in clear text, which is what IMSI catchers exploit
- 5G encrypts the Subscription Permanent Identifier (SUPI) with the home network's public key into a Subscription Concealed Identifier (SUCI) before sending it
- The Anti-Bidding Down Between Architectures (ABBA) parameter was added to make forced downgrades harder, and stronger temporary identifier generation was suggested but is still optional in the standard
- Design limitation examples: a phone always sends a SUCI when asked as it needs an identity to register, and a Registration Reject sent before authentication is still processed as no Non-Access Stratum (NAS) security context exists yet
- Implementation gap examples: using the null encryption scheme for the SUCI, predictable temporary identifiers, or modems being too permissive with certain messages
- Traditional IMSI catchers use jamming which is disruptive and easy to detect, modem fuzzing (sending malformed messages) assumes the attacker controls the whole network for a long time and usually crashes things, and conformance testing only runs in labs
- 5G-Shark instead uses the legitimate mobility procedure, so it can audit real phones on real commercial networks with minimal disruption

## Background

- Temporary identifiers (TMSI and GUTI, Globally Unique Temporary Identifier) were added to limit how often the IMSI is sent, but low refresh rates made them trackable too, and a fake base station can just ask for the IMSI directly
- The SUPI is usually just the IMSI, which is up to 15 digits made of the Mobile Country Code (MCC, 3 digits), the Mobile Network Code (MNC, 2-3 digits) and the Mobile Subscriber Identification Number (MSIN, up to 10 digits) which identifies the actual subscriber within that operator, it can also be a Network Access Identifier (NAI) in the form user@realm
- The SUCI is made of the SUPI type (IMSI or NAI), the home network identifier (MCC and MNC), a routing indicator which tells the network which server should handle it, the protection scheme ID, the ID of the home network public key used, and the scheme output which carries the concealed MSIN
- Only the MSIN is concealed, the MCC and MNC are still sent in clear text as the visited network needs them to route the request to the subscriber's home network
- There are 3 protection schemes: the Null-Scheme (scheme output is just the MSIN in clear text), Profile A (uses Curve25519) and Profile B (uses the NIST P-256 curve)
- Profiles A and B both use ECIES (Elliptic Curve Integrated Encryption Scheme), the home network's public key is stored on the SIM, and every time a SUCI is needed the phone generates a new one-time (ephemeral) key pair
- The phone does an elliptic curve Diffie-Hellman key exchange between its ephemeral private key and the home network's public key to get a shared secret, which is put through a key derivation function to get an AES-128 encryption key and an HMAC-SHA-256 key
- The MSIN is encrypted with AES-128 in counter mode and a MAC tag is calculated over the ciphertext, the scheme output is the ephemeral public key, the ciphertext and the MAC tag
- In the home network, the Subscription Identifier De-concealing Function (SIDF) uses the home network's private key with the ephemeral public key to get the same shared secret, checks the MAC and decrypts the MSIN
- As a new ephemeral key is used every time, the same SUPI produces a different SUCI on each registration, so an eavesdropper cannot recover the identity or link two SUCIs together
- Many deployments still default to the "Null-Scheme", which sends the SUCI unencrypted
- Equipment identifiers (IMEI and its software version IMEISV) are not protected by design
- 4G and earlier only use symmetric cryptography and have no public key infrastructure, so any downgrade to 4G strips the SUCI protection and the phone goes back to sending the IMSI in clear text
- Cell reselection is how a phone in idle (RRC_IDLE) or inactive (RRC_INACTIVE) state picks the best cell to camp on, unlike handover it is controlled by the phone rather than the network and does not require authentication, as it prioritises battery life over security
- Reselection can be inter-frequency (different frequency, same technology, which this attack uses), intra-frequency (same frequency) or inter-RAT (different Radio Access Technology, eg 5G to 4G)
- Each frequency has a priority, broadcast in System Information Blocks (SIBs), and the phone follows 3 rules:
- Higher priority: move to the candidate cell if its signal is above a high threshold
- Lower priority: only fall back if the current cell is poor and the candidate meets a minimum threshold
- Equal priority: rank the current cell by signal plus a bonus for staying (Rs = Qmeas,s + Qhyst) against the candidate minus an offset (Rc = Qmeas,c - Qoffset), and move if the candidate ranks higher
- As priorities come from unauthenticated broadcasts and decisions are based on signal measurements, a fake base station can advertise a high priority or boost its signal to get picked

## Methodology

### Threat model

- The goal is to harvest the permanent and temporary identifiers of anyone in range by running a fake base station (F-BS) that impersonates a commercial cell
- The F-BS must broadcast the same Mobile Country Code (MCC) and Mobile Network Code (MNC) as the real operator, know the operator's frequency plan (from SIB4 broadcasts, which list the priorities of neighbouring frequencies) and be strong enough to pass the reselection threshold
- The target is any unmodified phone in range following the 3GPP mobility standard (TS 38.304), with no user interaction needed and no warnings shown
- Only works when the phone is idle or inactive, they measured with "adb shell dumpsys" that phones were idle almost 70% of the time
- Exploits 3 design limitations: cell reselection, phones answering an unauthenticated IDENTITY_REQUEST, and phones processing an unauthenticated REGISTRATION_REJECT
- ![Figure 1 from the paper: the four phases of the fake base station attack, from decoding the legitimate cell's SIB4 and broadcasting a higher priority, through extracting the GUTI, IMSI or SUCI and IMEISV, to a Registration Reject that causes an infinite loop, denial of service or a downgrade to 4G]({{ '/assets/papers/5g-shark/figure-1.png' | relative_url }})

### Phase 1: Configuration

- The attacker picks the operator (MCC/MNC) and decodes SIB4 from the legitimate cell to find its frequencies and their priorities
- The F-BS is configured on the highest priority frequency, so it is not just a valid candidate but the preferred one

### Phase 2: Triggering reselection

- Unlike jamming or overshadowing (overpowering the real signal), the standard requires the phone to move to a higher priority cell as long as its signal is above the threshold, so low transmit power is enough, which may also avoid detection systems that look for signal strength anomalies
- 3 possible outcomes: the phone picks another legitimate 5G cell (F-BS signal too weak), the phone drops to a legitimate 4G cell (signal fluctuations, fixed by adjusting the priority), or the phone camps on the F-BS and starts registering

### Phase 3: Identifier extraction

- The phone sends a REGISTRATION_REQUEST containing its temporary identity (5G-GUTI), which the attacker logs
- The F-BS replies with an IDENTITY_REQUEST for the SUCI, pretending it does not recognise the GUTI
- If the Null-Scheme is used the SUCI contains the IMSI in plaintext, if Profile A or B is used it is properly encrypted, in which case the attacker can ask for the IMEI/IMEISV instead

### Phase 4: Rejection and downgrade

- The F-BS ends the connection with an unprotected REGISTRATION_REJECT, and the 5G Mobility Management (5GMM) cause code chosen (from TS 24.501) decides what the phone does next
- Depending on the code, the phone can delete its security context and re-register, abandon 5G and fall back to LTE (bidding down), or be blocked from service entirely (DoS)

### Implementation

- Built on srsRAN 5G (the gNodeB, ie the base station) and srsRAN 4G, with Open5GS as the core network, where the Access and Mobility Management Function (AMF) was modified to skip authentication and manipulate messages
- Hardware is a USRP X310 SDR connected over 10 GbE to an Intel NUC (i9-13900H, 32 GB RAM), with 2 transmit antennas, 2 receive antennas and a GPS antenna
- The modified AMF deliberately fails the GUTI lookup, sends an IDENTITY_REQUEST, logs the response, then sends a REGISTRATION_REJECT with the chosen cause, eg #10 "Implicitly De-registered" for a loop that keeps harvesting identifiers and downgrading, or #7 "5GS Services Not Allowed" for an immediate DoS
- It impersonates 3 operators (A, B and C) and rotates between them every minute using time-division multiplexing, one minute (58 s on average) was enough to trigger reselection, harvest identifiers and let phones reconnect to the real network
- It also logs supported security algorithms, disconnection timestamps, the subscriber's home network and the number of registration attempts, which allows tracking within the coverage area
- Network parameters (frequencies and SIB4) were collected passively with diagnostic tools like Keysight Nemo Handy and Samsung's service mode
- 7 phones were first tested on a lab 5G SA (Standalone) network with programmable SIM cards to check they actually support SA

### NSA attack

- 5G Non-Standalone (NSA) uses the 4G core for its control plane
- The F-BS broadcasts a new Tracking Area Code (TAC), so the phone sends a Tracking Area Update (TAU) request protected with keys from the real network
- The F-BS cannot verify it, so it rejects it with cause #10, which forces the phone by the standard to delete its GUTI and security context and re-attach
- The phone sends an Attach Request, the F-BS asks for the IMSI, and as 4G has no protection before authentication the phone sends it in clear text
- The Attach Request also reveals the phone's supported encryption and integrity algorithms, which can be used to fingerprint the device
- If the F-BS then goes silent, timer T3410 expires and the phone bars the cell and returns to the real network, but after T3402 (typically 12 minutes) it unbars the cell and comes back, so the extraction repeats

### SA attack

- Operator A gave its cells priority 5 out of 7, so an F-BS at 7 wins reselection immediately
- Operator B used the maximum priority of 7 everywhere, so the equal priority ranking applies, and instead of increasing power they degraded the real signal by shielding it with aluminium foil
- The phone sends a Registration Request with its 5G-GUTI, the F-BS asks for the SUPI, and the phone replies with a SUCI (or the IMSI in clear text under the Null-Scheme)
- When rejected with cause #91, the phone retried exactly 5 times, leaking its identity each time, then barred the cell and fell back to NSA, and after T3502 (12 minutes) it came back, trapping it in a loop

### Traceability

- The F-BS accepts a registration then releases the connection, forcing the phone to get a new 5G-GUTI from the real network, which is then captured on its next visit
- The GUTI from each REGISTRATION_REQUEST is mapped to the identity in the IDENTITY_RESPONSE in the same burst
- Step size is the median difference between a subscriber's consecutive GUTIs, as a percentage of the operator's effective range (the top 2 bits are fixed by the operator so only 16-25% of the 2³² space is used), a fully random allocator would give ~29%
- In SA where the identity is concealed, each subscriber's GUTIs are grouped by nearest value, starting a new subscriber only when a value is more than 1% of 2³² away
- Linkable share is the fraction of consecutive re-registrations that stay within 1% of 2³² of the previous GUTI

### Unauthenticated signalling

- A Galaxy S23 on operators A and B (SA) was sent each Registration Reject cause code and its behaviour recorded
- Codes were grouped by the 3GPP categories (subscriber identification, subscription, network/PLMN failures and invalid messages) and each behaviour was classed as a design limitation or an implementation gap

## Results

- NSA: every phone leaked its IMSI on every operator, which is a design limitation as NSA relies on the 4G control plane
- SA: the 3 phones that could connect to commercial SA concealed the SUCI correctly, except a Galaxy S23 on Operator A with a 2010 SIM card which sent its IMSI in clear text, the others could not connect to commercial SA at all as their bands were not supported, even though they worked in the lab
- Traceability (3742 GUTIs): Operator A (SA and NSA) and Operator B NSA moved in steps of at most 0.11% of their range with 84-96% of re-registrations linkable, so subscribers are easily tracked, Operator C was essentially random (28.9%, 7% linkable) and Operator B SA was in between (~50% linkable), so this is an implementation gap
- Most reject codes made the phone retry 5 times then fall back to LTE/NSA, losing SUCI protection, while #3, #6 and #7 pushed it down to UMTS (3G) with no data and only emergency calls, where #7 should have kept LTE (implementation gap)
- #11 and #73 caused an infinite retry loop that drains the battery instead of backing off, and #12 froze the S23's modem until airplane mode was toggled

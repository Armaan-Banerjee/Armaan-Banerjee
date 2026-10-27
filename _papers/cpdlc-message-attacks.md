---
title: "Sliding into the Flight Deck's DMs: Practical Message Attacks on CPDLC"
date: 2026-10-23
authors: "Mehdi Ziazi, Khalid Aleem, Harshad Sathaye, Martin Strohmeier"
paper_url: "https://www.usenix.org/system/files/usenixsecurity26-ziazi.pdf"
---

## Abstract

- CPDLC is an alternative to VHF communication like ACARS with messages for altitude changes, heading adjustments, free-text and frequency handovers
- CPDLC does not implement encyprtion and uses seucirty through complexity
- The paper shows that CPDLC can be hijacked and a large scale DOS can completley diable the funcionaltiy of CPDLC for all aircraft in radio range
- CPDLC-GS is a SDR based CPDLC implementaiton that can inject uplink messages to issue fake messages

## Introduction

- CPDLC (Controller-Pilot Data Link Communciations) lacks encyptions, authemtication or cryptographic message identity
- An attack could have extreme impacts, in a DOS attack, fallback to redundant/ replacement systems may not be immediate especially in oecanic airspace or high-workload phases, or could navigate aircraft to dangerous situations like towards other aircraft or restricted airspace
- CPDLC are perceived as authoritative and so pilots have a high level of trust in this system

## Background

- Analog voice radio has previously served as the main method of controller-pilot communication
- voice channels are highly congested, leading to transmission delays, overlaps and radio blocking and readback erros and miscommunication is common
- CPDLC provides structured text based messaging protocol
- CPDLC is currenlty deployed using FANS-1/A+ over ACARS or ATN B1 over VDL Mode 2
- Fans-1A+ uses the existing ACARS network using HF, VHF or satcom links, it provides lower data throughput but has high compatability and low infrastrucutre needs
- ATN B1 is used in high-density airspace particulalry over Europe, it uses VD Mode 2 offering high data rates and some message integrity using Protected Mode CPDLC and integrats directly with radar and transponder systems and has been mandatory over 28,500 feet in European airspace
- CPDLC over ATN B1 follows the OSI based protocol stack
- ![Table 1 from the paper: the PM-CPDLC ATN-B1 protocol stack, from VDL Mode 2 at the physical layer up to CPDLC messages at the application layer]({{ '/assets/papers/cpdlc-message-attacks/figure-1.png' | relative_url }})
- CPDLC over ATN B1 uses the 117.975 to 137 MHz band and uses VDL2 which uses a single carrier channel using differential 8 Phase Shift Keying at 10.5 thousand symbnols/second, each transmission begins with ramp up and preamble, and followed by a scrambled error-corrected payload
- THe bitsream is interleaved with Reed Solomon and scrambled using a pseudo-noise suequence
- Advanced Linkc Control (AVLC) is used as a High-Level Data Link Control protocl for VHF flinks and does sequence numbering and labels frames I (data transfer), U(error recovery), S (basic flow control) etc
- VDL2 uses XID (Exchange Identification) on the AVLC U frames to negotiate parameters such as frequencies, capabilities, and ATN (Aeronatuical Tellocommunications Network) routing information
- ![Figure 2 from the paper: simplified CPDLC connection establishment between air traffic control and an aircraft, from link setup through to the CPDLC application session]({{ '/assets/papers/cpdlc-message-attacks/figure-2.png' | relative_url }})
- X.25 operates on AVLC and establishes virtual circuits, and support basic flow control
- End System-to-Intermediate System (ES-IS) is a disovery protocol to find the Network Service Access Point (NSAP) address from X.25 Data Terminal Euipment (DTE) address
- ConnectionLess Network Protocol (CLNP) X.233 provides NSAP based addressing, routing, error reports and Protocol Data Unit lifetime control
- Inter Domain Routing Protocol (IRDP) is a routing information exhcange like BGP wihting CLNP, each domain chooses its routing policy, and IRDP ensures that all border iintermediate systems calculate inter domain paths which satisy those policies
- Connection Orinted Transport Protocol (COTP) X.224 is a connection-oriented transport services, similar to TCP but outside OSI model

## Denial of service

### Malicious AVLC Frame Reject (FRMR) Injection

- If a received frame is unacceptable, the FRMR response sends a protoocol error with a copy of the rejected control field, sequence numbers and error flags
- ![Sequence diagram of a malicious AVLC FRMR injection: the attacker sends an FRMR to the aircraft, which replies with an AVLC UA, causing ATC to send an FRMR and reset the link]({{ '/assets/papers/cpdlc-message-attacks/figure-3.png' | relative_url }})
- The aircraft will respond with an AVLC UA (unumbered Acknolwedgement), which causes an AVLC Frame Reject (FRMR) from the ATC system, causing a session teardown
- Targetting the COTP induces a disconnet request and causing the transport alyer to abort
- Targetting IRDP exchanges (Open, update, keepalive) can push the finite state machine into the closed state, cutting of routing association between aircraft and the group
- X.25 Receive ready can also be targetted

### Broadcast DoS

- This targets VDDL mode 2 and the behaviour of the Datalink Entity (DLE) and its communication witht he Link Management Enityt(LME)
- Normally the VDL management starts a DLE which is controlled by the corresponding LME
- When receiving an AVLC Unnumbered Disconnect (U DISC), the DLE transitions to the Asychronous Disconnected Mode (ADM) and notifies the LME
- If AVLC U DISC is transmitted using the broadcast ICAO address 0xFFFFFF all aircraft associated with the sender address will disconnect
- This means all aircraft have to restart the ATN stack from the physical layer

### Control FLow Injection

- Incrementing the AVLC I/ X.25 Data Sender Sequence, this is achieved through injecting control flow frames like X.25 Receive Ready or ES-IS Hello PDU, which are correct payloads, but end up incrementing the AVLC sender sequence number, therefore when the aircraft receives these frames (as they are sent to the aircraft), it updates its expected sequence state, as the ATC did not see these frames, it has a wrong sender sequence number, which causes a link reset and an FRMR
- Disrupting the Virtual Circuit: X.25 Reset Request, resets the sender and receiver sequence numbers to 0, X.25 Clear Request tears down the virtual circuit, COPT Disconnect Request aborts the connection at the transport layer and IDRP CEASE PDU transition the IRDP FSM to closed position, all of these trigger FRMR from ATC

### Malformed Frame Injection

- At higher layers, malformed frames must have the correct lower layers to work, eg higher layer manipulation will require knowledge of the negotiated CLNP parameters such as error report flag and selected COTP protocol class
- Malformed COTP or IRDP messages that bypass error reporting, like those with all zero ATN checksum, are automatically rejected, causing an X.25 RR and causigna. link state reset

![Figure 11 from the paper: the race condition for state-based DoS, where the attack succeeds only if the attacker's malicious frame reaches the aircraft before the legitimate frame from ATC]({{ '/assets/papers/cpdlc-message-attacks/figure-4.png' | relative_url }})

## Evil twin

- CPDLC-GS was used to implement a rogue ground station that issued valid CPDLC messages, operating as a finite state machine and going up the ATN stack
- IF would broadcast a Ground Station Information Fram (GSIF) advertising standard VDLM2 with D8PSK modulation at 136.975 MHz, as well as a series of AVLC U frames to negotiate link-layer parameters
- They impelemnted both AVLC and X.25 to support many packets types and ACARS encasulation
- ES-IS (End System to Intermediate System) protocol was then used
- NASP addresses were ther inferred from teh DTE addresses, and using these and IRDP session was intiated
- The ground station then established transport connection with COTP, COPT messages are encapsulated in compressed CLNP frames with correct ATN checksum
- After transport layer is set up, the session layer is used to receive the Aircraft's CPDLC Logon request, which includes the aircraft's callsign and airport pair
- Once CPDLC session is establsihed, the ground station transmits uplink messages to the aircraft
- The rouge ground station could issue a UM79 PROCEED DIRECT, diverting the aircraft away toward a waypoint that could potentially be unsafe due to conflicting movements from other aircraft or restricted airspace
- A UM21 CLIMB TO or UM20 DESCEND TO cleears aircraft to an alttitude which could be assigned to another aircraft, which could lead to TCAS advisories
- A malicious step descent can clear an aircraft to descend which is normally done for fuel efficiency, but could be used to put the aircraft in severe adverse weather or opposite traffic flow
- A UM80 CLEARED VIA can be dangerous on approach as it causes to aircraft to not comply with nomral traffic flow
- It can also clear aircraft for an early descent whcih can put it on the approach path without clearance, increasign chance of collision
- MAINTAIN speed instruction can slow an aircraft in hgih density airspace, risking seperation breaches
- A false DIVERT TO alternate airport causes operation disruption
- UM169 (free text instructioin) can cause confusion by saying the opposite of what an existing ATC cleared them for
- A UM169 can also cause a fabricated emergency forcing crew to execute emergy checklists and undermine trust

## Real world constraints

- The rouge ground station and aircraft must be far enough away from the legitimate ground station for transmission to be undetected , otherwise it would respond with an AVLC U FRMR
- Other radar surveillance is also used like Primary/secondary radar (SSR), ADS-B give infomration to ATC about aircraft speed, however both can be spoofed and Dosed as well
- VHF readio is the primary fallback which is also vulnerable to spoofing, a malicious CPDLC can redirect cre3ws to attacker controlled frequeuncies, which could delayd ATC detection
- TCAS is used to issue traffic and resolution advisories which stops most of these attacks being catastrophic, however is also vulnerable to spoofing

---
title: "A Gmail-Based Phishing Detection Prototype for Nigerian Fintech Emails Using Sender Checks and BiLSTM Classification"
date: 2026-10-23
authors: "Gideon Francis Oghie, Uche Emmanuel Unoke"
paper_url: "https://arxiv.org/pdf/2609.28305"
---

## Abstract

- Gmail browser extension that combines sender-domain and url checks with a bidrectional long short-term memory model
- The BiLSTM classifier used 8943 text messages, and got 4,308 true negatives, no false positives, 1 false negative, and 4,634 true positives

## Methodology

- 1,000 phishing examples were created before cleaning to represent domanins, language and platform
- Nigerian Pidgin was applied to some of the emails
- Email text was converted to tokens from a vocabulary of 10,000 terms, each sequence being padded of truncated to 500 tokens
- The embedding layer initalised 100 dimensional GloVe vectors ( Global vectors for Word Representation which is an unsupervised machine learning model (not a neural network ) which turns words into vectors) which fed 2 BiLSTM with 128 and 64 units
  ![Diagram of an LSTM cell showing the forget, input and output gates acting on the previous cell state, previous hidden state and current input]({{ '/assets/papers/phishing-detection-extension/figure-1.png' | relative_url }})
- The model was trained with binary cross-entropy loss, at a learnign rate of 0.001, training used batches of 64 for up to 20 epochs
- A Chrome Manifest V3 extension implemented the detection process, a MutationObserver detected when a message was opened, the extension extracted the sender address, subject, body text, and hypderlinks, the results for the rule based layer (which includes sender domain verifier, fintech sender database and url typosquatting analyser) was fused with ML layer decision to get a score
- An exact match with a listed sender domain received an interenal status of verified, and a domain with a Levensthein edit distance was marked as suspicious, this stage did not check email headers, or DKIM or DMARC auth
- A BiLSTM score of >-0.7 or unverified sender score of >= 0.8 received PHISHING verdicet, and when unverified sender with score form 0.6 to 0.8 present was also marked PHISHPING when there was a suspicious link, remaning cases were marked as WARNING

---
title: "Context Language Models"
date: 2026-10-23
authors: "Rulin Shao, Shannon Zejiang Shen, Junjie Oscar Yin, Yuetai Li, Minheng Wang, Hamish Ivison, Radha Poovendran, Nathan Lambert, Teng Xiao, Mike Lewis, Wen-tau Yih, Luke Zettlemoyer, Pang Wei Koh"
paper_url: "https://arxiv.org/pdf/2609.37725"
---

## Basic idea

- Previouly context has been append only for the actual model, the harness has been responsible for managing context
- The harness has multiple options, it can summaris, delete, etc
- Harnesses have previously been configured to start compaction at a certain limit eg 75% or after x tool calls
- Some harnesses use subagents to avoid growing the context
- This however ignores taht some documents are more useful than others, which the harness may ignore, which shouldnt be summarised away or compressed
- Making the context available as a file and telling the LLM to manage the context by itself introduces modest perfomrance increases of ~10%
- However, by editing the context in. this way, it invalidates the KV cache after the point of change, say the model decides to change the world prize to award in the sentence The man won the prize for being the best, KV cache becomes invalidates after the word "the" as they attend to a vector no longer present
- The paper proposes suffix cache reuse, like how the KV Cache is reused for "The man won the", the cache after the edited section is also the same. just shifted, therefore can be reused
  ![Figure 4 from the paper: standard serving re-prefills everything after an edit, while Suffix Cache Reuse reuses the cache for the unchanged text after the edit]({{ '/assets/papers/context-language-models/figure-1.png' | relative_url }})

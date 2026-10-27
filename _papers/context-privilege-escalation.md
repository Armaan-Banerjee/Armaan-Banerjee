---
title: "What's in Your Agent's Context? Context Privilege Escalation Attacks against AI Agent Harness"
date: 2026-10-23
authors: "Zichuan Li, Jian Cui, Ashley Chen, Xiaojing Liao, Luyi Xing"
paper_url: "https://arxiv.org/pdf/2609.01222"
---

## Abstract

- 2 types of attacks, Message-Role Context Privilege Escalation (M-CPE) where attacker controlled content is incorporated into a higher privileged role and Cross Scope Context Privilege escalation (X-CPE) where the attacker finds a way to persist their input and affect future sessions

## Introduction

- To mitigate indirect prompt injection, labs have designated priority roles for different types of input, system message is highest priority wheras tool calls are lower priority. (Example from OpenAi: system, developer, user, assistant and tool)
- M-CPE is where less trusted, low privileged role can affect higher privileged context
- X-CPE is where persistence is added
- CoRA was devloped to identify context sources based on privilege roles, peparing context and running agent harness and to run proof of context by sending malicious messages
- ![Fig. 1 from the paper: an example Codex CLI agent context, listing each source (built-in instructions, execution policy, skills, project memory, environment, user prompt, model reasoning, tool output) with its privilege role and scope]({{ '/assets/papers/context-privilege-escalation/figure-1.png' | relative_url }})

## Attacks

- Every agent loads files to give information about project history at launch, these are often given the highest priority, can be exploited by malicious instruction from tool outputs writing into these memory files
- Different agents have different have different methods to search for memory files, claude and codex start at cwd and load all through upper layer directtories until reaching a project boundary, gemini will load all gemini.md from subdirectories. Can be attacked by having a malicious pull request place a malicious gemini.md deep in the directory structure, making it hard for human to spot, this is automatically read into the agent's context irrespective of pull requests's status which can influence the agent
- Agents can load memory files during runtime, when claude edits files in a direcotry it searches for claud.md and loads it into context, can be attacked by having a third party or tool call have a claude.md component within its subdirectory, when claude loads these filess it is at a higher priority
- Agents often load skills into context at runtime, agents have different ways of doing it some load from .agents/skills, all load at high priority, some agents also only search for skills in their subdirectory like claude, wheras others search for all skill files, can be exploited by modifying a skill.md file

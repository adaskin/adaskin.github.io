---
layout: post
title:  "DeepSeek and scientific research"
date:   2025-01-27 07:00:00 +0300
categories: AI
tags: AI DeepSeek
comments: true
---


After using DeepSeek for **free** and seeing all the hype and the worries of big companies, I have to say I am feeling a little happy.
I think opensource and openscience communities-whose code and books&research papers used to train tools that are closed and very expensive for the most of the world-have taken their revenge by DeepSeek.

The reasoning steps/thoughts in DeepSeek R1 reminds me of a very inteligient librarian in light novels who may be just a very small step short to become a famous inventor/innovator/thinker.
It can review most topics better than most review articles and with a little push it gives briliant concepts for future research directions.

For the system programming course next semester, it has given me a lot of good suggestions for how to integrate DeepSeek into the teaching ([see example prompts in syllabus](https://github.com/adaskin/bil222-sysprog25?tab=readme-ov-file#use-of-ai-gpt-gemini-deepseek-etc)). For now, I am not sure how these tools can be employed in learning and teaching without restraining youth potential or if it would be beneficial. But, I am happy that it is free and everybody can access this without paying $200 a month.

Next, I am waiting for advanced chips from China to crash the chip prices...

## Update on running locally
I ran  deepseek-r1 32b distilled version with ollama on local-CPU which has required  22 GB RAM.
32B version is definitely not as smart as the full version although it gives very similar answers. 
For example, the full version can go beyond the available knowledge 
and think through topics and possibilities with you, 
while the smaller versions simply cannot.

To run on local version, 
- you can directly download the source code or pretrained models from [https://huggingface.co/deepseek-ai](https://huggingface.co/deepseek-ai). 
- Or download and run through [https://ollama.com/library/deepseek-r1](https://ollama.com/library/deepseek-r1) 
which is simpler if you want to set-up a server.

Note that running ollama on docker requires extra a few GB RAM because of additionally running a container.
Also note that full version of 471B parameters requires more than  GPU with 400GB vram.


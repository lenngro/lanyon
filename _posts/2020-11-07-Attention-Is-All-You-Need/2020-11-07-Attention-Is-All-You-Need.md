---
layout: post
title: "Attention Is All You Need"
author: Lennart Grosser
date: 2020-11-07
category: "paper-summary"
description: "A summary of the paper 'Attention Is All You Need' by Vaswani et al."
tags: ["machine learning", "deep learning", "attention", "self attention", "self-attention", "transformer"]
---

> This post is a summary of the paper [Attention Is All You Need, Vaswani et al., 2017](https://arxiv.org/abs/1706.03762). The paper describes a novel sequence transduction model, the transformer, an encoder-decoder model that works only through attention mechanisms. Parts of the original paper may be left out.


## Introduction: History of approaches to sequence modeling and transduction problems
In the introduction the authors give a brief overview of previous approaches to sequence modeling and transduction problems, i.e. recurrent neural networks such as [long short-term memory][1] or [gated recurrent units][2]. They describe the **fundamental problem** that those model architectures inhibit, that is the **constraint of sequential computation**. Recurrent neural networks factor computatation along the symbol positions of the input and output sequences, yielding a sequence of hidden states $$h_{t}$$ for the $$t$$th sequence step, as a function of the previous hidden state $$h_{t-1}$$ and the input $$x_{t}$$ for the position $$t$$. This dependency on sequential inputs and computation inherently prevents  parallelization. Although different tricks exist that speed up the computation, the underlying problem can't be solved.

The authors propose a novel model architecture, the _transformer_, that solely **relies on attention mechanisms** to learn global dependencies between input and output and **overcomes the parallelization problem**. The transformer achieves new starte of the art translation quality.

## Background: Previous solutions to reduction of sequence computation and the role of self-attention
The background section describes how previous approaches (Extended Neural GPU, ByteNet, ConvS2S) tried to reduce the amount of sequential computation. All of those approaches relied on convolutional neural networks computing representations in parallel for all input and output positions. However, the number of computation steps that are required to relate input to output increases with the distance between two positions (linearly for ConvS2S, logarithmically for ByteNet). In contrast, the proposed **transformer** architecture **defines a constant number of required operations to process a sequence** (however, _possibly_ at the **cost of reduced prediction quality** due to **averaging attention-weighted positions**). This is one of the very key points of the proposed architecture (enabling complex sequence modeling without the need for sequential computation).

## Model Architecture: The Transformer In Detail
Similar to previous architectures, the transformer has an encoder-decoder structure:
![Transformer Architecture](/assets/images/2020-11-07-Attention-Is-All-You-Need/transformer-model-architecture.png)
The left side shows the encoder that encodes the input sequence $$(x_{1}, ..., x_{n})$$ to a sequence of continous representations $$z = (z_1, ..., z_n)$$, the right side shows the decoder that decodes the output of the encoder combined with the output (embeddings) to an output sequence $$(y_1, ..., y_m)$$. The decoder works [_auto-regressive_](https://en.wikipedia.org/wiki/Autoregressive_model): for each outputs the model consumes all previously generated symbols as additional input. As can be seen from the image, the transformer architecture consists of encoder (decoder) blocks that can be stacked (--> Nx) upon each other to create a more and more complex model.
### Encoder & Decoder Blocks
#### Encoder
The vanilla transformer architecture uses $$N=6$$ encoder blocks where each block consists of 2 sub-layers, a multi-head self-attention mechanism and a position-wise fully connected feed-forward network. Around each sub-layer a [residual connection](https://stats.stackexchange.com/questions/321054/what-are-residual-connections-in-rnns) is employed. Afterwards follows a layer normalization. Final output of each sub-layer: $$LayerNorm(x + Sublayer(x))$$. All embedding layers as well as all sub-layers in the model produce outputs of the dimension $$d_{model} = 512$$.
#### Decoder
The decoder is structured similarly to the encoder. The difference is that the decoder applies an additional multi-head attention layer that works on the output of an encoder block. Again, residual connections are employed around each sub-layer followed by layer normalization. For all positions, the multi-head attention that works on output embeddings masks all subsequent positions so that a prediction for step $$i$$ depends only on previously seen positions. Additionally, the output embeddings are shifted right by one position so that the first prediction is the first symbol (without shifting the first symbol would be known during the first prediction...not what we want). The vanilla transformer applies also $$N=6$$ decoder blocks.

[1]: https://www.bioinf.jku.at/publications/older/2604.pdf
[2]: https://arxiv.org/abs/1412.3555

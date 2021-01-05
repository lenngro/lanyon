---
layout: post
title: "Summary: Attention Is All You Need"
author: Lennart Grosser
date: 2020-11-07
category: "paper-summary"
description: "A summary of the paper Attention Is All You Need, Vaswani et al., 2017. The paper describes a novel sequence transduction model, the transformer, an encoder-decoder model that works only through attention mechanisms. Parts of the original paper may be left out."
tags: ["machine learning", "deep learning", "attention", "self attention", "self-attention", "transformer"]
---

> This post is a summary of the paper [Attention Is All You Need, Vaswani et al., 2017](https://arxiv.org/abs/1706.03762). The paper describes a novel sequence transduction model, the transformer, an encoder-decoder model that works only through attention mechanisms. Parts of the original paper may be left out.

## Introduction: History of approaches to sequence modeling and transduction problems
In the introduction the authors give a brief overview of previous approaches to sequence modeling and transduction problems, i.e. recurrent neural networks such as [long short-term memory][1] or [gated recurrent units][2]. They describe the **fundamental problem** that those model architectures inhibit, that is the **constraint of sequential computation**. Recurrent neural networks factor computation along the symbol positions of the input and output sequences, yielding a sequence of hidden states $$h_{t}$$ for the $$t$$th sequence step, as a function of the previous hidden state $$h_{t-1}$$ and the input $$x_{t}$$ for the position $$t$$. This dependency on sequential inputs and computation inherently prevents  parallelization. Although different tricks exist that speed up the computation, the underlying problem can't be solved.

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
### Attention
The paper starts the attention section by giving a short explanation of what attention in deep learning is and how it works. It describes attention as mapping a query and multiple key-value pairs to an output, that is the weighted sum of the values. The weight of a value is computed as the _compatibility function_ of the query and the value's key.

The transformer architecture uses attention in two ways: scaled dot-product attention and multi-head attention where multi-head attention internally uses scaled dot-product attention.

#### Scaled Dot-Product Attention
The scaled dot-produt attention is computed as follows:

$$Attention(Q, K, V) = softmax(\frac{QK^{T}}{\sqrt{d_k}})V$$

What are $$Q$$, $$K$$, $$V$$ and $$d_k$$ here? $$Q$$ ($$K$$, $$V$$) are multiple queries $$q$$ (keys $$k$$, values $$v$$) stacked to a matrix. These matrices are the result of multiplying the input $$X$$ with a set of learned matrices $$W_Q$$, $$W_K$$ and $$W_V$$. Therefore, all of $$Q$$, $$K$$ and $$V$$ are weighted representations of the input $$X$$. By multiplying them, attention achieves that all parts $$x$$ of the input $$X$$ are evaluated against each other and pairs of coherent parts can be identified. This enables understanding of relationships between different parts of the input without the need for sequential computation.

The dimension of $$Q$$, $$K$$ and $$V$$ can be controlled through the dimension of $$W_Q$$, $$W_K$$ and $$W_V$$. This allows creating more or less complex representations, depending on the need or resources.

The dot-product in the transformer architecture is additionally scaled by $$\frac{1}{d_k}$$ because without scaling the values of the dot product might grow into regions where the softmax function would create extremely small gradients.
#### Multi-Head Attention
Multi-head attention is a way to project the queries, keys and values into higher dimensions by learning separate attention _heads_. An attention head is basically the result of a single scaled dot-product attention. Multiple of those heads are then concatenated and weighted through an additional weight matrix $$W_O$$. In the vanilla transformer, 8 attention heads are computed, from which $$d_{model} / 8 = d_k = d_v = 64$$ results. Why is this needed? Computing multiple attention heads allows _attending to information from different representation subspaces at different positions_.
#### Connecting Encoder to Decoder through Attention
In an encoder-decoder block, the queries $$q$$ come from the previous decoder while the keys and values come from the encoder's output. By taking the keys and values from the encoder and not from the previous decoder, the decoders can also access the information in the input embeddings instead of only relying on the output embeddings. Within an encoder, queries, keys and values originate from the output from the previous encoder. This means an encoder can access all positions and representations from the previous encoder.
### Feed Forward Networks
In both encoder and decoder blocks, a _position-wise feed forward network_ (FNN) follows the multi-head attention. A FNN consists of two linear transformations that add another level of complexity to each block:

$$FNN(x) = max(0, xW_1 + b_1)W_2 + b_2$$

The FNN is called position-wise because the transformations (the weight matrices) are the same across all positions (however different in different layers). $$max(0, a)$$ is a [ReLU activation](https://en.wikipedia.org/wiki/Rectifier_(neural_networks)). The paper describes this layer as equivalent to a two convolutions with kernel size 1.
### Positional Encoding
Although the transformer architecture works without sequentially processing the input, the information that is given by the sequence itself ("_a_ happened **after** _b_") should still be taken advantage of. The transformer achieves this by adding a _positional encoding_ to the input. The positional encoding is a vector containing values that should provide information about the relative or absolute position of a part of the input. The architecture uses the following functions to compute the encodings:

$$PE(pos, 2i) = sin(pos(10000^{i / d_{model}}))$$

$$PE(pos, 2i+1) = cos(pos(10000^{i / d_{model}}))$$

The authors explain this particular choice of positional encoding functions because for any fixed offset $$k$$, the position $$PE_{pos, 2i + 1}$$ can be represented as a linear function of $$PE_{pos}$$ which makes it easy for the model to learn this relationship. The functions allow for longer sequence lengths than seen during training.

## More On Self-Attention
The paper gives more insight on why the authors chose to use an architecture solely relying on self-attention over previous recurrent or convolutional layers. They define three criteria to compare the methods:
1. total computational complexity per layer
2. amount of computation that can be parallelized / min. number of sequential operations required
3. path length between long-range dependencies in the network

The third point is relevant for two reasons: 1. learning long-range dependencies between different parts of the input is crucial for effective sequence transduction problems and 2. the length of the path a forward (and backward) signal has to traverse is linked to the level of difficulty the model has to overcome to learn such a dependency (longer paths are harder to learn).

The comparison of the different approaches are given below:

![comparison](/assets/images/2020-11-07-Attention-Is-All-You-Need/comparison.png)

From the table several things can be seen:
* self-attention layers are faster than recurrent layers when the sequence length is smaller than the representation dimensionality $$d$$
* by considering only the $$r$$ surrounding words of a given position, the computational performance could be further improved (maximum path length would then be $$O(n/r)$$).
* convolutional layers are generally even more expensive than recurrent layers

Another advantage of self-attention layers is increased interpretability.

## Remaining Sections
The remaining three sections of the paper describe training set-up, results and the conclusion and are left out here because they mostly don't give much more insight on the internals of the model's architecture. However, the reader is advised to have a look in the [original paper](https://arxiv.org/abs/1706.03762).

However, some fairly interesting visualizations are given in the appendix that visualize how different heads in the multi-head attention learn different tasks. One of those is shown below:

![multi-head-attention](/assets/images/2020-11-07-Attention-Is-All-You-Need/multi-head-attention.png)

The picture shows two different attention heads. Obviously both heads learned to solve different tasks: 1. the left attention head (red) learned short-range relationships between parts of the input (which word relates to which other words in its close preceeding words),  2. the right attention head (green) visualizes more distant relationships (starting from more or less one word in each subsentence).


[1]: https://www.bioinf.jku.at/publications/older/2604.pdf
[2]: https://arxiv.org/abs/1412.3555


[Byte Latent Transformers](https://arxiv.org/pdf/2412.09871) are a novel architecture proposed by Meta researchers, intended as a tokenizer-free approach operating on byte-level data without a performance compromise. Tokenization is a rote approach to information digestion. It abuts the end-to-end training approach of Large Language Models (LLMs), biased by the provided vocabulary in its compression of textual information leading to [separate tokenization approaches for domain-specific performance](https://arxiv.org/pdf/2402.14903). 

Researchers present a neoteric solution to the previously prohibitive costs associated with training on byte-level data at scale. Byte Latent Transformers dynamically group bytes into patches, eliminating the need for a fixed vocabulary. Patching schemes create contextualized groupings of byte-level information with corresponding compute allocation, as opposed to tokenizer-based approaches which allocate the same amount of compute to every token.

Light-weight encoder and decoder modules map groups of bytes into latent patch representations operated on by a global latent transformer, responsible for reasoning and predicting the next sequence of latent text. The primary computational expense for both tokenization- and patch-based models are the requisite steps through the prediction Transformer. For the Byte Latent Transformer, this is the quantity of patches needed to encode the data, a controllable variable allowing efficient allocation of compute as needed.

We’ll examine the different patching schemes presented in the paper below, before diving into the model architecture.

# Patching Schemes

An illustration of the authors’ presented patching schemes can be seen below.

<p align="center" width="100%">
  <img src="/Images/patching_schemes.png" width="100%"
</p>

# Architecture

## Local Encoder

### Embeddings

## Global Latent Transformer

## Local Decoder

##### Thoughts here or on separate page

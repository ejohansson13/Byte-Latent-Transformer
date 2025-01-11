[Byte Latent Transformers](https://arxiv.org/pdf/2412.09871) are a novel architecture proposed by Meta researchers, intended as a tokenizer-free approach operating on byte-level data without a performance compromise. Tokenization is a rote approach to information digestion. It abuts the end-to-end training approach of Large Language Models (LLMs), biased by the provided vocabulary in its compression of textual information leading to [separate tokenization approaches for domain-specific performance](https://arxiv.org/pdf/2402.14903). 

Researchers present a neoteric solution to the previously prohibitive costs associated with training on byte-level data at scale. Byte Latent Transformers dynamically group bytes into patches, eliminating the need for a fixed vocabulary. Patching schemes create contextualized groupings of byte-level information with corresponding compute allocation, as opposed to tokenizer-based approaches which allocate the same amount of compute to every token.

Light-weight encoder and decoder modules map groups of bytes into latent patch representations operated on by a global latent transformer, responsible for reasoning and predicting the next sequence of latent text. The primary computational expense for both tokenization- and patch-based models are the requisite steps through the prediction Transformer. For the Byte Latent Transformer, this is the quantity of patches needed to encode the data, a controllable variable allowing efficient allocation of compute as needed.

We’ll examine the different patching schemes presented in the paper below, before diving into the model architecture.

# Patching Schemes

Patching schemes are responsible for dividing byte-level data into contextualized conceptual information groups. They demarcate boundaries between patches, creating ingestible chunks of knowledge for the model to internalize and reason over. They offer a more flexible approach to information division than tokenization, which relies on a static set of tokens drawn from a fixed vocabulary. Patching can adapt to new text, as its decision-making is only predicated on the underlying byte features. An illustration of the authors’ presented patching schemes can be seen below.

<p align="center" width="100%">
  <img src="/Images/patching_schemes.png" width="100%"
</p>

## K-Strided Patching

The most straightforward approach to patching is fixing an invariable number of bytes to every patch, highlighted in red in the image above. It’s inherently easy to implement, with k bytes constituting a singular patch and a stride of k bytes for the next patch. However, it eliminates the advantages of patching, with every grouping of bytes treated as equally important and inconsistent patching of identical byte sequences potentially confusing the global transformer. 

## Whitespace Patching

Another intuitive approach, highlighted in green in the above illustration, whitespace patching divides text by space-like bytes (any non Latin character, Arabic digit, or UTF-8 continuation byte), a natural linguistic boundary. It affords consistent patching of byte sequences, but fails to adapt to all domains and offers equal attention to short and longer words.

## Entropy Patching

Entropy is a measure of informational uncertainty. Using a small byte-level autoregressive language model trained on the same data as the overall Byte Latent Transformer, researchers can derive entropy estimates to inform patch boundaries. If the byte-level model has high uncertainty over the next byte prediction, the model denotes the start of a new patch, quantifying patch boundaries according to their associated prediction difficulty. In the image seen below, every byte above the red dotted line (global entropy threshold) denotes the start of a new patch.

<p align="center" width="100%">
  <img src="/Images/global_entropy_threshold.png" width="100%"
</p>

### Monotonicity

Rather than employing a global entropy threshold, a second approach utilized a relative entropy threshold, where new patches would begin with bytes breaking the monotonically decreasing entropy of the previous patch. Determining an optimal global entropy threshold can be difficult and researchers instead allowed for noticeable entropy differences between sequential bytes to distinguish between patches.

# Architecture

## Local Encoder

### Embeddings

## Global Latent Transformer

## Local Decoder

##### Thoughts here or on separate page

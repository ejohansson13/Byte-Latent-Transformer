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

### Embeddings

Text needs to be projected to a vector space to propagate through a neural network. Tokenization-based models achieve this trivially, mapping tokens 1:1 with a vector embedding. Operating at the byte level required a new approach. Researchers designed hash n-gram embeddings. As a first step, this required each individual byte to be embedded. Next, the byte-grams were created where, for each individual byte, the preceding n-bytes were collected, with n ranging from three to eight. 

Each n-gram was mapped to an index in a fixed-size embedding table via the Rolling Polynomial Hashing function, with embedding tables corresponding to the n-gram length. Given the vector embedding of the singular byte and the hash embeddings for its corresponding n-grams, the augmented embedding, serving as input to the local encoder, could be calculated as the sum of the byte embedding with each hashed n-gram embedding. This summation is illustrated below.

<p align="center" width="100%">
  <img src="/Images/augmented_embedding_summation.png" width="100%"
</p>

Operating at the byte-level, hash n-gram embeddings embed individual bytes before summing the vector embeddings corresponding to their hashed n-gram complements, encoding contextual information into each augmented embedding.

## Local Encoder

<p align="center" width="100%">
  <img src="/Images/local_encoder_diagram.png" width="90%"
</p>

The local encoder consists of stacks of alternating Transformer and cross-attention layers. The Transformer layers attend to the byte embeddings, self-attending up to the current byte position. Cross-attention infuses the patch representations with information from the byte embeddings by employing the patch representations as query vectors, while the self-attended byte embeddings serve as the key and value vectors.

Patches are computed as a pre-processing operation occurring during dataloading. They serve as placeholders, denoting boundaries between bytes, iteratively distilling information from the byte embeddings during steps through the local encoder. They operate on, and up to, the bytes within their boundaries. However, since the augmented byte embeddings contain preceding information via their hash n-gram embeddings, patches receive implicit contextual information crossing patch boundaries.

The augmented byte embeddings propagate through the Transformer layers, refining their information through the attention mechanism. This information is repetitively passed on to the patch representations which serve as the hidden states in the diagram at the beginning of [this section](#local-encoder). After propagating through all of the layers in the local encoder, the final layer’s output serves as the latent patches input to the global latent transformer.

The specific architecture for the Transformer layer can be seen below, a pipeline of normalization, self-attention, normalization, and the feed-forward network utilizing the [SwiGLU](https://arxiv.org/pdf/2002.05202) activation function. 

<p align="center" width="100%">
  <img src="/Images/transformer_layer.png" width="90%"
</p>

If you’re not familiar, the feed-forward network with the SwiGLU gated linear unit activation function is illustrated below. It’s a compilation of linear operations, projecting the byte embeddings to a higher dimension, one of those projections proceeding through a [Sigmoid Linear Unit](https://arxiv.org/pdf/1606.08415) before the two paths are summed, and the result is projected back down to the original dimension through another linear operation.

<p align="center" width="100%">
  <img src="/Images/FFN_SwiGLU.png" width="90%"
</p>

## Global Latent Transformer

## Local Decoder

##### Thoughts here or on separate page

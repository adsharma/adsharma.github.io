# Towards Explainable AI

Large language models consume vast amounts of training data and spit out billions of parameters that have hard to pin down meanings. Embeddings in a higher dimensional space form the basis on which other technologies such as transformers and retrieval augmented generation (RAG) are built.

However, there is no universally accepted semantic space over which ideas can be expressed. Every model creates its own inscrutable parameters which doesn't mean anything for another model.

For applications such as semantic databases and collaborative filtering, having a common embedding vocabulary is a necessity.

## Matryoshka Embeddings

Some progress was made in 2022 towards building this common language. This [paper](https://arxiv.org/abs/2205.13147) is highly influential and has changed how smaller models (~300 million params) understand language and how databases store the data. Example [blog post](https://blog.vespa.ai/combining-matryoshka-with-binary-quantization-using-embedder/).

Specifically it allows you to run training once and then compute embeddings that are much smaller in size (64 bytes vs 1024 bytes), that allows an implementer to make quality vs performance/storage trade-off without having to run training N times. In other words you can think of MRL as doing hierarchical clustering over a common semantic space.

However, if you take one of the language models that uses Matryoshka models (I used `mxbai-embed-large-v1`) and embed the names of members of congress, the numbers it spits out mean nothing. They have nothing in common.

That lead to realization that Matryoshka based models have a common understanding of the language, but have no understanding of the underlying concepts.

## Combining hierarchical clustering of language and human knowledge

Some concepts have meanings that are independent of the language. For example former US presidents `Barack Obama` and `Donald Trump` may have something in common in Japanese and Spanish. But each of them embeds the corresponding words in a different embedding space.

So how do we align these Matryoshka language models in a common semantic space?

## Wikidata to the rescue

Turns out that the largest publicly available knowledge graph has all of these concepts. But they're expressed through unreasonably large [json dumps](https://dumps.wikimedia.org/wikidatawiki/entities/) or behind [sparql](https://query.wikidata.org/), a language few industry practitioners speak.

However, there are people who improve the situation significantly. I found someone who computes pagerank on the wikidata graph every couple of weeks and makes it [freely availble](https://danker.s3.amazonaws.com/index.html).

## Duckdb, Pandas and Nvidia RAPIDs

After some investigation, I found that duckdb, pandas and Nvidia RAPIDs allow cheap computation on a 3 year old laptop.

Duckdb compressed a 35GB json dump into 2.6GB. That's a significant improvement. Details coming in another blog post.

You can compute a high pagerank only subgraph of wikidata (containing about 660k nodes and 8 million edges) using the dump.

Running the [Leiden Community Detection](https://en.wikipedia.org/wiki/Leiden_algorithm) on the graph gives you a 3 level hierarchical clustering of the data in a few minutes.

Nvidia RAPIDs library comes with GPU optimized algorithms that make such fast run times on cheap hardware possible.

## Construct a universal semantic space

After running Leiden, I was able to compute the following

```
select * from leiden_c_flat using sample 10 order by part3, part2, part1;

┌──────────┬───────┬───────┬───────┬────────────────────┐
│  vertex  │ part1 │ part2 │ part3 │      combined      │
│  int64   │ int64 │ int64 │ int64 │       uint64       │
├──────────┼───────┼───────┼───────┼────────────────────┤
│   885870 │   120 │     1 │     0 │     18133977333760 │
│ 75577586 │   103 │     4 │     0 │     70917059510272 │
│  3652407 │   166 │    10 │     0 │    176706018869248 │
│  1978817 │    70 │    26 │     1 │  72515299630120960 │
│ 10131941 │    14 │    28 │     1 │  72550305117831168 │
│   491556 │   172 │     0 │     3 │ 216173528349999104 │
│  2650907 │    98 │    11 │     3 │ 216366776086364160 │
│  7485229 │    25 │    13 │     4 │ 288459191647010816 │
│ 31898933 │    88 │    32 │     4 │ 288793724076490752 │
│  3682267 │   136 │    18 │     6 │ 432662878956224512 │
├──────────┴───────┴───────┴───────┴────────────────────┤
│ 10 rows                                     5 columns │
└───────────────────────────────────────────────────────┘
```

part3, part2, part1 were computed by Leiden by recursively running the algorithm and then `combined` was computed by using the following formula:

```
(part3 :: UINT64 << 56) | (part2 :: UINT64 << 44) | (part1 :: UINT64 << 32) | (rowid :: UINT64 << 16)
```

Note that `rowid` is the order in which edges appeared in the input. It's correlated with time, but for now lets consider it random.

What we've achieved is to take 660k most important concepts in wikidata and embed them into a 64 bit space, where numbers mean something.

For example, if you compare the `combined` ids for the two former presidents ([Q76](https://www.wikidata.org/wiki/Q76) and [Q22686](https://www.wikidata.org/wiki/Q76)), you'll notice that they have the same part3, part2 and very similar higher bits.

```
select vertex, format('{:x}', combined), part3, part2, part1 from leiden_c_flat where vertex=22686 or vertex=76;

┌────────┬──────────────────────────┬───────┬───────┬───────┐
│ vertex │ format('{:x}', combined) │ part3 │ part2 │ part1 │
│ int64  │         varchar          │ int64 │ int64 │ int64 │
├────────┼──────────────────────────┼───────┼───────┼───────┤
│  22686 │ 30000210dde0000          │     3 │     0 │    33 │
│     76 │ 300000f10f30000          │     3 │     0 │    15 │
└────────┴──────────────────────────┴───────┴───────┴───────┘
```

## What next?

There are attempts to compute an embeddings that combine both concepts. References I could find: [HybridRAG](https://arxiv.org/html/2408.04948v1) and [Joint Alignment](https://github.com/dki-lab/joint-kb-text-embedding).

However, I could not find anyone who sets up a periodically recomputed jointly aligned kb-language embedding.

Once there is a publicly accessible embedding, other industry practitioners can compete on information retrieval performance based on a widely accepted industry standard embedding.

When you have excellent [NER](Named Entity Recognition) tools such as [GliNER](https://github.com/urchade/GLiNER) that can extract knowledge from natural language text, if they can also embed those extracted entities in a widely accepted common embedding space, it unlocks a huge amount of potential.

## Applications

You don't like the amount of outrage coming at you from various social networks and news headlines from corporate media? Instead of trying to remove content based on keywords, you can now write a filter saying I don't want to see anything related to concepts between (`0x30000210dde0000` and `0x300000f10f30000`) and live your life in peace.

With a bit of [extra work](https://paperswithcode.com/method/transe) perhaps you can write a filter that says I don't want to hear anything negative about concept X or person Y.

There is a rich ecosystem of extracting knowledge from text. One is called NER (Named Entity Recognition) and an emerging technology space called Relationship Extraction. While LLMs [can do this](https://github.com/circlemind-ai/fast-graphrag), I believe it can be done in a much cheaper way using smaller models at scale.

What these ecosystems are missing is a common universal embedding. I hope this work encourages some converation around arriving at one.

I'm happy to publish the code and the embeddings computed if anyone is interested.
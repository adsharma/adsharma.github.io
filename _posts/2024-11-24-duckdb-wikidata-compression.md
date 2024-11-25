# Compressing Wikidata Using DuckDB

Wikidata publishes a 40GB blob called [latest-truthy.nt.bz2](https://dumps.wikimedia.org/wikidatawiki/entities/latest-truthy.nt.bz2) every two days. This is the largest publicly available human knowledge base as a graph. For someone interested in following changes
to this graph, we present a way to consume *8x* less network bandwidth.

The file contains "statements" known to be true.  Let's examine the first node in the dump: Q31 (Belgium).

A statement can be a label of a node 

```
Q31 P9100 Belgium  # P9100 = "Github topic" for Q31. Label is Belgium.
```

or its relationship with other nodes. 

```
Q31 P36 Q239 # Brussels is the capital of Belgium. 
```

as you can guess, P36 is a "property", specifically means the "capital of" and Q239 is the city of Brussels.

If you were to construct a graph from this data, you care only about rows of the form `<Qxx Pxx Qxx>`. But wikidata doesn't
publish those subsets. To quantify the benefit I ran some experiments.

## Experiment

Download the [latest dump](https://dumps.wikimedia.org/wikidatawiki/entities/20241120/) which is published every 2 days based on the filenames.

```
curl -L -O -C - https://dumps.wikimedia.org/wikidatawiki/entities/latest-truthy.nt.bz2
```

You can wrap this in a loop if you prefer the download to resume automatically.

### Extracting signal from the data

While the download was still going on, I ran [truthy.py](https://gist.github.com/adsharma/71bd591ea5d4242ec4e250e7fc7d20d1) in parallel. It processed the useful parts of the input and spit out a `truthy.db` which was only about ~4.3GB in size on disk.

```
bzcat latest*.gz2 | ./truthy.py
```

The script prints out progress every 100k lines processed.

After 8 hours:

```
$ ls -l truthy.db
-rw-rw-r-- 1 ubuntu ubuntu 7747153920 Nov 25 09:16 truthy.db
$ du -sh truthy.db
4.3G    truthy.db
```

After removing duplicates and sorting:

```
$ ls -l clean.db
-rw-rw-r-- 1 ubuntu ubuntu 5004341248 Nov 25 15:46 clean.db
$ du -sh clean.db
2.8G    clean.db
```

The size on disk is a bit smaller because of lz4 compression. In other words, we would be downloading only 5GB instead of 40GB for savings of 8x! Savings could be 16x if `clean.db` is also compressed with lz4.

### Request to wikimedia folks

Can you also please run the attached script and publish truthy.db, which is much more convenient to download? The idea is that the
labels don't change often. Q31 remains Belgium. But the relations do.

Publishing deltas to the graph would be an added bonus. For now, just the subset will do.

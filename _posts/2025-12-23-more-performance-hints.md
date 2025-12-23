
This is some commentary on [Performance Hints](https://abseil.io/fast/hints.html) doc from Jeff Dean and Sanjay Ghemawat from someone who has written a lot less code and was a hot headed heretic in the 90s and whose career overlapped with Jeff and Sanjay a few times over the last 25-30 years.

I don't disagree with anything Jeff and Sanjay say in the doc. Most of their advice is sound and will result in faster code without sacrificing readability.

## NUMA and HyperThreading

These are two words that you don't hear much about in the doc. Talk to any performance engineer who worked at a CPU vendor during the time period referenced above and they'll tell you these are the two most important optimization techniques.

A leading distributed database got 2x the throughput by optimizing for NUMA alone in the last 5 years. NUMA became mainstream with AMD CPUs circa 2005. HyperThreading became mainstream circa 2000 with Intel CPUs.

Certainly benchmark your memory latency beyond the L3 cache more aggressively vs memory bandwidth. You'll learn a lot more about your hardware, OS and will likely have a larger impact on general purpose C++ code. Of course, for highly parallel and tuned special purpose code, memory bandwidth could be more important.

## Columnar Storage vs Protobuf/Thrift

The doc talks about Protobuf, which has been reported to consume 10% of Google's CPU fleetwide. As they correctly advise, copying things around and backward compatibility have a cost that should be optimized.

Missing: Apache Arrow and the ecosystem around it including Arrow Flight RPC do a much better job of shuttling data around the datacenter. They have been around for a [decade](https://www.dremio.com/blog/the-origins-of-apache-arrow-its-fit-in-todays-data-landscape/) and deserve a place in any such widely read performance doc.

This performance crime is particularly acute in the database world where people either put a Protobuf wrapper around an otherwise efficient wire protocol, either via another service or an in-process API.

If you are a performance focused engineer at Trillion dollar companies where Protobuf/Thrift are ubiquitous, it's easy to be blind to what's happening elsewhere in the industry. Massive $100B+ companies are getting built around these ideas.

## Hardware/Software Co-design

Google has a few examples of such co-design. TPUs and JAX, Pixel phone and Android. But it hasn't been the case with general purpose server hardware. While many Hyperscalers make their own ARM CPUs, such server designs were not possible due the long and complicated history of x86 CPUs.

This probably led to a view inside these companies that the companies with smaller market caps should constrain their hardware to match the massive software codebases that were getting written.

## Brawny Core Consensus

There were plenty of sympathizers of this view inside those CPU companies, which became natural allies. Thus the Brawny Core consensus was born, along with a minimum viable CPU core design for the data center - 2GHz clock, 2MB/core.

This was a direct result of some of the data structures choices in libraries such as Abseil. Rewriting them for wimpy cores was considered a software cost not worth it.

There was a period of 5-10 years where GPUs/TPUs were also seen as niche designs that violated the Brawny Core consensus.

Even the Nvidia B200 which deviates significantly from the Brawny Core consensus [spiritually](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#the-benefits-of-using-gpus) is pretty close. SM clock speed is ~2GHz and 0.85MB of L2 cache/SM.

Ironwood (TPUv7) is a significant departure. The TensorCores run at 700MHz-1GHz. Cache MB/TFLOP is about half of the B200. **0.036 MB / TFLOPS** vs **~0.015 MB / TFLOPS**.

And programmers have figured out software abstractions to program these chips by effectively combining scripting languages and compiled code and new programming paradigms (SIMT, cuTile, systolic arrays).

## Conclusion

Jeff and Sanjay are among the most accomplished programmers in my generation. They do a great job of describing their craft. They've shared large chunks of their professional work as open source code. Let's celebrate their contributions. At the same time, there are significant new dimensions to upcoming hardware and software. Columnar databases, new LLM designs and more that involve principled Hardware/Software Co-design.

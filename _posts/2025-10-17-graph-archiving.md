# Packaging Graphs: A Tale of Two Sparse Cousins

Everyone knows `tar` files! Perfect for shuttling data between filesystems like ext4 and xfs, preserving structure, ownership, and timestamps for seamless transfers—ideal for backups or migrations. ISO files, meanwhile, are the mount-and-go champs, letting you access contents directly without unpacking, like a virtual disk for quick reads or software distribution. In the graph world, Apache GraphAr and graph-std ([GitHub](https://github.com/adsharma/graph-std)) mirror these paradigms, each leveraging Compressed Sparse Row (CSR) internally to keep sparse graph data lean and traversable.

GraphAr, with its "ar" nod to tar archives, is built for portability, bundling graph data into a directory of Parquet files for easy database ingestion. graph-std, akin to an ISO, offers a mountable, query-friendly structure for CLI tools like DuckDB and Python to explore graphs without heavy lifting. Both envision a [lake house](https://graphar.apache.org/blog/graphs-openlakehouse) built on top of this tech.

## GraphAr: The Tar of Graphs

Apache GraphAr is your go-to for moving massive graphs between systems. Starting from a database—say, Nebula or Ladybug - you load vertices and edges, then serialize them into a directory of Parquet files: one Parquet file per property (e.g., `vertex_property_name.parquet`, `edge_property_weight.parquet`) plus metadata. This bundle, easily zipped or tarred, is a self-contained artifact ready for bulk loading into another database like JanusGraph. It’s like tarring a directory for rsync: no extraction mid-transit, just a portable, schema-rich payload for cross-system hops.

## graph-std: The ISO for Graph Exploration

graph-std, detailed in its [karate_csr example](https://github.com/adsharma/graph-std/tree/main/karate/karate_csr), is the ISO equivalent—mount it and query instantly. Designed for CLI simplicity, it uses DuckDB to convert diverse inputs (CSVs, SQL dumps, edge lists) into a standardized bundle for tools like ladybugdb. Its output is lean: three core Parquet files (four if you include the optional mapping):
- **metadata.parquet**: Schema and CSR details.
- **indices.parquet**: CSR edge destinations, stored in a single Parquet file with multiple row groups for chunked access.
- **indptr.parquet**: CSR row pointers, defining where each node’s edges start and end.
- **node_mapping.parquet** (optional): Vertex relabeling; defaults to an identity mapping if absent.

This setup lets you "mount" the graph by querying directly with DuckDB or Ladybug. It’s like browsing an ISO without unpacking—ideal for data scientists poking at graphs in Jupyter or CLI-driven analytics.

## Key Differences: Structure and Style

Despite their CSR foundation, GraphAr and graph-std diverge in execution:
- **YAML** vs **ladybug catalog**: graph-std prefers keeping metadata about the table in the system catalog of the database. This design is inspired by DuckLake's design.
- **Naming**: GraphAr uses "vertex" and "edge," aligning with graph theory. graph-std goes with "node" and "edge," nodding to Pythonic simplicity (think NetworkX).
- **File Structure**: graph-std consolidates into three (or four) Parquet files, using row groups for internal sharding—compact and CLI-friendly. GraphAr requires a directory of Parquet files per property, plus a manifest, favoring schema flexibility but increasing complexity.

## Unify or Diverge? The Big Questions

Should GraphAr and graph-std merge into one standard? Their purposes—GraphAr for data transfer, graph-std for direct queries without ingesting—suggest keeping them separate. Unification is also appealing, but we need to solve some design decisions. Middle ground: align on small stuff: adopt "vertex" over "node" for consistency across docs and make splitting parquet files optional. For massive graphs, such as a 100 GB dataset, graph-std’s single-file approach (e.g., one `indices.parquet`) could become a bottleneck. Sharding into, say, 100 1 GB files (`indices_part_{i}.parquet`) with metadata stitching them together is a fix, but DuckDB’s `EXPORT DATABASE` doesn’t split natively. You’d need a `COPY TO` loop with `FILE_SIZE_BYTES` checks to shard dynamically—a scripting hassle graph-std could address with a built-in CLI flag.

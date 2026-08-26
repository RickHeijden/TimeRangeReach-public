# TimeRangeReach-public

## Public executable

`TimeRangeReachPublicExe` is the command-line executable developed as part of the thesis *Temporal GeoTopological Reachability Queries:
Algebra, Indexing, and Evaluation*. It loads a temporal geosocial graph, constructs the selected reachability index/algorithm, executes
a query file, and reports the Boolean result of each query together with aggregate performance statistics.

The executable contains the following 13 public methods:

| Family                    | Methods                                          |
|---------------------------|--------------------------------------------------|
| HR_Index baselines        | `hr2d`, `hr3d`, `hrsi`                           |
| Online traversal baseline | `ot`                                             |
| LIS indexes               | `lis`, `lis-fem`, `lis-seg`, `lis-3d`, `lis-3dl` |
| LIE indexes               | `lie`, `lie-seg`, `lie-3dl`                      |
| SPATI index               | `spati`                                          |

Use `--help` or `-h` to display the complete command-line interface. More information regarding these methods is available in the thesis.

### Running queries

```bash
./TimeRangeReachPublicExe \
  --method lis-3dl \
  --graph <graph-prefix> \
  --queries <queries.qry> \
  --time-dimension cc
```

`--graph` takes a shared file prefix rather than a complete filename. The executable expects these two files at that prefix:

- `<graph-prefix>-entity.txt`: the graph entities, including which entities are spatial and their coordinates.
- `<graph-prefix>-graph.txt`: the graph edges and their temporal information.

For example, if the files are `datasets/sample/sample-entity.txt` and `datasets/sample/sample-graph.txt`, pass
`--graph datasets/sample/sample`. Both files must exist; do not include either suffix in the argument. The `graph-builder-public` explained
below outputs both of these files when applied to a dataset.

`--queries` selects one query file, and `--time-dimension` selects conjunctive-continuous (`cc`) or disjunctive-continuous (`dc`) temporal
semantics. Every provided query set is used with both `cc` and `dc`; the thesis describes the remaining experimental protocol. The
executable prints one `TRR_RESULT` row per query followed by aggregate timing and index-size statistics.

### Snapshot loading threads

The optional `--snapshot-load-threads <N>` argument controls how many worker threads load snapshot data. Its default value is `0`, which
means **auto**: the executable uses the machine's reported hardware concurrency, but never more threads than there are snapshots to load.
For example, on a 16-thread machine with three snapshots, auto uses three loader threads. A positive value requests that many threads,
also capped by the number of snapshots.

### Runtime requirements

The provided executables target 64-bit Linux (`x86-64`) and require glibc 2.38 or newer and a libstdc++ version that provides
`GLIBCXX_3.4.34`. The `fbw` graph-builder dataset additionally requires the system time-zone database (`tzdata`) with
`Australia/Sydney` available.

## Datasets

The experimental datasets are derived from publicly available graph datasets used in previous work. The original datasets are not included
in this repository and must be obtained separately from the sources below.

| Dataset    | Thesis datasets                    | Paper             | Original dataset     | Download checksum                      |
|------------|------------------------------------|-------------------|----------------------|----------------------------------------|
| Foursquare | `foursquare`                       | [Geosocial paper] | [Dataset collection] | [SHA-256](checksums/foursquare.sha256) |
| Gowalla    | `gowalla`                          | [Geosocial paper] | [Dataset collection] | [SHA-256](checksums/gowalla.sha256)    |
| WeePlaces  | `weeplaces`                        | [Geosocial paper] | [Dataset collection] | [SHA-256](checksums/weeplaces.sha256)  |
| Yelp       | `yelp`                             | [Geosocial paper] | [Dataset collection] | [SHA-256](checksums/yelp.sha256)       |
| Facebook   | `fbw`, `fbl`                       | [HR-Index paper]  | [Facebook Links]     | [SHA-256](checksums/facebook.sha256)   |
| Amazon     | `amazon-3`, `amazon-4`, `amazon-5` | [HR-Index paper]  | [Amazon dataset]     | [SHA-256](checksums/amazon.sha256)     |

[Geosocial paper]: https://openproceedings.org/2025/conf/edbt/paper-13.pdf

[HR-Index paper]: https://dl.acm.org/doi/10.1145/3589272

[Dataset collection]: https://seafile.rlp.net/d/6fa3703e57234b6d9831/

[Facebook Links]: https://socialnetworks.mpi-sws.org/data-wosn2009.html

[Amazon dataset]: https://www.comp.hkbu.edu.hk/~db/book/community_search.html

The provided `graph-builder-public` executable transforms the downloads into the temporal geosocial graph representation used by the
experiments. It augments the source graphs with the required temporal and spatial information. The exact generation procedure is fixed
within `graph-builder-public`; further details are provided in the thesis.

For Foursquare, Gowalla, WeePlaces, and Yelp, `<input>` is a prefix: pass the shared path without `-entity.txt` or `-graph.txt`. For
example, if the downloaded files are `foursquare-entity.txt` and `foursquare-graph.txt`, pass `/path/to/foursquare`. For Facebook and
Amazon, pass the complete source filename: `facebook-links.txt` and `Amazon0601.txt`, respectively. In every case, `<output>` is a prefix;
the builder writes `<output>-entity.txt` and `<output>-graph.txt`.

Each linked checksum file verifies the downloaded input file or pair used by the builder. Run `sha256sum --check` from the directory
containing those files, for example:

```bash
cd /path/to/downloaded/foursquare
sha256sum --check /path/to/TimeRangeReach-public/checksums/foursquare.sha256
```

#### Example usage graph-builder-public

```text
Usage:
  ./graph-builder-public fbw <input> <output>
  ./graph-builder-public fbl <input> <output>
  ./graph-builder-public amazon-3 <input> <output>
  ./graph-builder-public amazon-4 <input> <output>
  ./graph-builder-public amazon-5 <input> <output>
  ./graph-builder-public foursquare <input> <output>
  ./graph-builder-public gowalla <input> <output>
  ./graph-builder-public weeplaces <input> <output>
  ./graph-builder-public yelp <input> <output>
```

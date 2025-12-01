# SPQR tree file format `.spqr` v0.1

## Basics
* When a line contains a #, then the rest of it is ignored
* Lines always start with a letter indicating the type of the line
* Following the line type, each line is a list of space-separated identifiers that can be arbitrary ASCII
* Identifiers must be globally unique. For example, if we declare a component G0, then we cannot declare another component G0, and also no node G0, ...
* Things must be declared before they are used. For example, the G-line for declaring a component must be before all lines that refer to it

## Design decisions
* Plain text to be bioinformatics-ready
* Sometimes redundant to allow easier parsing
* Trade-off between human readability and small file size. To optimize file size, use e.g. gzip
* The overall structure is inspired by GFA

## Line types

A `...` in examples specifies that the previous identifier type can occur multiple times.

### H-line

Each `.spqr` file has exactly one header line.
It specifies the file format version, as well as a URL pointing to the format specification at that version.

`H v0.1 https://github.com/sebschmi/SPQR-tree-file-format [optional extra data]`

### G-line

Declare a connected component with its contained nodes.
A G-line also acts as declaration for its contained nodes.

`G <component name> <node name> ...`

### N-line

Declare extra data for a graph node.
This is entirely optional.
If no metadata is present, it suffices to declare the nodes in a G-line.

`N <node name> <extra data>`

### B-line

Declare a block (i.e. a 2-connected component) with its contained nodes.
The `<component name>` is the name of the component containing this block.

`B <block name> <component name> <node name> ...`

### C-line

Declare a cut node that connects a set of blocks.
The set of blocks must be exactly the blocks that contain the node.

`C <node name> <block name> ...`

### S/P/R-line

Declare an SPQR-tree node of type S, P or R with its contained nodes.
The `<block name>` is the name of the block containing this SPQR-tree node.

`S <S-node name> <block name> <node name> ...`

`P <P-node name> <block name> <node name> ...`

`R <R-node name> <block name> <node name> ...`

### V-line

Declare an SPQR-tree edge between a pair of SPQR-tree nodes.
The SPQR-tree edge is between the two SPQR-tree nodes specified by the two `<S/P/R-node name>`s.
Within these SPQR-tree nodes, the SPQR-tree edge connects to a virtual edge in their skeletons.
This virtual edge is specified by its endpoints, the graph nodes specified by the two `<node name>`.

Note that different SPQR-tree edges may share a pair of graph nodes within some SPQR-tree node.
In this case, the SPQR tree is typically defined to contain parallel virtual edges.

`V <SPQR tree edge name> <S/P/R-node name> <S/P/R-node name> <node name> <node name>`

### E-line

Declare a graph edge between a pair of graph nodes (i.e. a Q-node).
The `<S/P/R-node name>` and the `<block name>` specify the SPQR-tree node and the block that contains this edge.
The `<node name>`s are the endpoints of the edge.
After one space after the last node name, there can be an arbitrary string (except for # and newline characters).
This may be used to add e.g. metadata to the edge.

`E <edge name> <S/P/R-node name> <block name> <node name> <node name> [optional extra data]`

## Format for extra data

Any extra data is a space-separated list of data items.
A data item has the format `key:type:value`.
Valid types are:

Type | Description | Example
-----|-------------|--------
`i` | Integer | `abundance:i:15` `weight:i:-4`
`f` | Float | `abundance:f:3.14` `x:f:1e-3` `y:f:-inf`
`d` | Sign of a bidirected edge incidence in mathematical format (see below) | `E E0 P0 B0 N1 N2 N1:d:+ N2:d:-`
`dgfa` | Sign of a bidirected edge incidence in GFA format (see below) | `E E0 P0 B0 N1 N2 N1:dgfa:+ N2:dgfa:+`
<nobr>`s`</nobr> | String without spaces | `seq:s:ACTGTGAACC`
<nobr>`s:<len>`</nobr> | String of length `<len>` bytes (spaces allowed) | `comment:s:16:SPQR trees rock!`
`b64` | Base64 encoded binary blob | `data:b64:U1BRUiB0cmVlcyByb2NrIQ==`

### Reserved keys

Key + Type | Description | Example
-----------|-------------|--------
`seq:s` | A DNA, RNA or protein sequence associated with a node or edge | `seq:s:GGCTA`

### Bidirected edge signs in mathematical format

In mathematical format, bidirected graphs as interpreted as having undirected edges (see e.g. [Medvedev et al. 2007](https://link.springer.com/chapter/10.1007/978-3-540-74126-8_27)).
A walk through such a bidirected graph changes sign at each node.

### Bidirected edge signs in GFA format

In GFA format, bidirected graphs are interpreted as having directed edges.
When a walk enters a node with a `+`, it must leave it through an outgoing edge with a `+`.
Implicitly, each `E` edge encodes two edges in this interpretation, one of which is in the opposite direction.
For example, an edge `E E0 P0 B0 N1 N2 N1:dgfa:+ N2:dgfa:-` implicitly also represents the edge `E E0 P0 B0 N2 N1 N2:dgfa:+ N1:dgfa:-`.
See the [GFA1 specification](https://gfa-spec.github.io/GFA-spec/GFA1.html#l-link-line) or the [GFA2 specification](https://gfa-spec.github.io/GFA-spec/GFA2.html#edge) for more details.

## Incomplete example

```spqr
# Header
H v0.1 https://github.com/sebschmi/SPQR-tree-file-format # The header line specifies the version of the file format and a URL pointing to a description of the format at this version

# 1-connected components
G G0 N0 N1 N2 N3 N4 N5 N6 # Declare a component G0 containing graph nodes N0-N6
N N3 seq:s:GGCTA weight:f:3.14 # Attach extra data to graph node N3

# 2-connected components
B B0 G0 N0 N1 N2 N3 N4 # Declare a block B0 inside component G0 containing graph nodes N0-N4
B B1 G0 N0 N5 # Declare a block B1 inside component G0 containing graph nodes N0 and N5
B B2 G0 N0 N6 # Declare a block B2 inside component G0 containing graph nodes N0 and N6
C N0 B0 B1 B2 # Declare that N0 is a cut node that connects blocks B0, B1 and B2

# 3-connected components
S S0 B0 N0 N1 N2 # Declare an S-node inside B0 containing graph nodes N0, N1 and N2
P P0 B0 N0 N1 # Declare a P-node inside B0 containing graph nodes N0 and N1
R R0 B0 N1 N2 N3 N4 # Declare an R-node inside B0 containing graph nodes N1, N2, N3 and N4
V V0 S0 P0 N0 N1 # Declare a virtual edge V0 connecting SPQR-nodes S0 and P0 between graph nodes N0 and N1
E E0 P0 B0 N1 N2 # Declare a graph edge E0 inside SPQR-node P0 and inside block B0 and between graph nodes N1 and N2
E E1 P0 B0 N2 N0 N2:d:- N0:d:+ seq:s:AAGATA # Declare a graph edge E1 with extra data
```

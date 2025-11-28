# SPQR tree file format `.spqr`

## Basics
* When a line contains a #, then the rest of it is ignored
* Lines always start with a letter indicating the type of the line
* Following the line type, each line is a list of space-separated identifiers that can be arbitrary ASCII
* Identifiers must be globally unique. For example, if we declare a component G0, then we cannot declare another component G0, and also no node G0, ...
* Things must be declared before they are used. For example, the G-line for declaring a component must be before all lines that refer to it

## Design decisions
* Plain text to be bioinformatics-ready
* Sometimes redundant to allow easier parsing

## Line types

A `...` in examples specifies that the previous identifier type can occur multiple times.

### G-line

Declare a connected component with its contained nodes.
A G-line also acts as declaration for its contained nodes.

`G <component name> <node name> ...`

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
The `<node name>`s specify the virtual edge within the connected SPQR-tree nodes pertaining to this SPQR-tree edge.

`V <S/P/R-node name> <SPQR-tree node name> <node name> <node name>`

### E-line

Declare a graph edge between a pair of graph nodes.
The `<S/P/R-node name>` and the `<block name>` specify the SPQR-tree node and the block that contains this edge.
The `<node name>`s are the endpoints of the edge.

`E <edge name> <S/P/R-node name> <block name> <node name> <node name>`

## Incomplete example

```spqr
# 1-connected components
G G0 N0 N1 N2 N3 N4 N5 N6 # Declare a component G0 containing graph nodes N0-N6

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
E E0 P0 B0 N0 N1 # Declare a graph edge E0 inside SPQR-node P0 and inside block B0 and between graph nodes N0 and N1
```

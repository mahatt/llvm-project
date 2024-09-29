# LLVM Backend
## Selection DAG
* Build DAG
  + Who builds DAG ?
  + How DAG is built?
    - Build DAG Nodes from IR, visit Each IR and Create DAG Nodes
    https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/SelectionDAGBuilder.cpp#L1320-L1345
  + ` visit(I.getOpcode(), I);`  dispatches to IR Instruction specific dispatch.
    https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/SelectionDAGBuilder.cpp#L1375-L1385

  + What happens to IR after DAG creation ?
* Optimize by Combine DAG
    https://github.com/mahatt/llvm-project/blob/879d70f1686dd936a22bd08592d91a70f9504059/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L2006-L2027
* Legalize
  + Types
  + Operations
  + https://github.com/mahatt/llvm-project/blob/879d70f1686dd936a22bd08592d91a70f9504059/llvm/lib/CodeGen/SelectionDAG/LegalizeDAG.cpp#L5926-L5936
## Machine IR
## MC Inst

## Inline ASM

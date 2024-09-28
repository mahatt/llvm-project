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
* Legalize
  + Types
  + Operations
## Machine IR
## MC Inst

## Inline ASM

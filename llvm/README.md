# LLVM Backend
## Code Generation 
* Code generation Passes
  https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/CodeGen.cpp#L19-L25
## Selection DAG
* Prepare for DAG
  https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/CodeGenPrepare.cpp#L561-L570
* Build DAG
  + Who builds DAG ?
  + How DAG is built?
    - Build DAG Nodes from IR, visit Each IR and Create DAG Nodes
    https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/SelectionDAGBuilder.cpp#L1320-L1345
  + ` visit(I.getOpcode(), I);`  dispatches to IR Instruction specific dispatch.
    https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/SelectionDAGBuilder.cpp#L1375-L1385

  + What happens to IR after DAG creation ?
* Optimize DAG with DAG Combiner
  https://github.com/mahatt/llvm-project/blob/879d70f1686dd936a22bd08592d91a70f9504059/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L1000-L1010
  
* Legalize
  + Types
    https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/LegalizeTypes.cpp#L201
    https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/LegalizeTypes.cpp#L245-L295
  + Operations
    https://github.com/mahatt/llvm-project/blob/879d70f1686dd936a22bd08592d91a70f9504059/llvm/lib/CodeGen/SelectionDAG/LegalizeDAG.cpp#L5926-L5936
* ISEL
  + Pattern Matching
    https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/Target/X86/X86ISelDAGToDAG.cpp#L5077-L5080

  + Machine Scheduler
    - https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/MachineScheduler.cpp#L424-L476
## Machine IR
## MC Inst

## Inline ASM

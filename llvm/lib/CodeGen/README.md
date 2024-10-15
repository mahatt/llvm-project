
# CODE GENERATION 
* Reads llvm-ir, generates DAG
* Legalizes Type "generically"
* Legalizes operation "generically"
* Optimizes DAG with Combiner "generically"
* Scheduler
## DAG Combiner
Runs from

  https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L1729-L1735

Node Visit/Traversal Algo

  https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L1753-L1790

Visiting Individual Combination Patterns

+ DAGCombiner::combine
  - If DisableGenericCombines is false, do DAGCombiner::visit(SDNode *N)
    It is Generic Combinbers, so all generic Nodes ISD nodes are handled here.
    https://github.com/mahatt/llvm-project/blob/85ae1e9b5e1f800172ac0fa2bdb7f297cd8d8ce4/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L1836C9-L1836C38
    * Vector Operation Combination
      + DAGCombiner::visitINSERT_VECTOR_ELT();
      + DAGCombiner::visitEXTRACT_VECTOR_ELT();
      + DAGCombiner::visitBUILD_VECTOR();
          - Case All Operands are undef
          - Case single element is replicated  splat aka broadcast
          - 
      + DAGCombiner::visitCONCAT_VECTORS();
      + DAGCombiner::visitEXTRACT_SUBVECTOR();
      + DAGCombiner::visitVECTOR_SHUFFLE();
      + DAGCombiner::visitSCALAR_TO_VECTOR();
      + DAGCombiner::visitINSERT_SUBVECTOR();
  - calls X86TargetLowering::PerformDAGCombine
  - https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/Target/X86/X86ISelLowering.cpp#L58116
  - calls PromoteIntBinOps,....
  - TargetLoweringInfo defines how operation is handled X86TargetLowering::getOperationAction(IDS::opcode, ValueType), return value is Action like TargetLowering::Expand
 
+ DAG Operation Driver, Combiner is executed 4 times, For each CombineLevel.
  https://github.com/mahatt/llvm-project/blob/85ae1e9b5e1f800172ac0fa2bdb7f297cd8d8ce4/llvm/include/llvm/CodeGen/DAGCombine.h#L15-L19
  It is driver from
  https://github.com/mahatt/llvm-project/blob/85ae1e9b5e1f800172ac0fa2bdb7f297cd8d8ce4/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp#L918-L1100
  
+ DAG Legalizer
  - Entry Point `SelectionDAG::LegalizeOp` `SelectionDAGLegalize::LegalizeOp` 
  - https://github.com/mahatt/llvm-project/blob/4d0cbc7f7933902fa011becf88e300d66adc75d9/llvm/lib/CodeGen/SelectionDAG/LegalizeDAG.cpp#L955-L985
  

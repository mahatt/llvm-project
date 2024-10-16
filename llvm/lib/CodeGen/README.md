
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
      + DAGCombiner::visitBUILD_VECTOR();
          - Why BUILD_VECTOR
            + https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L523-L529
            + Built from Constant lit Values, Constant FP nodes etc. 
          - Combination Cases
            - All Operands are undef
            - Single element is replicated  splat aka broadcast
              * (build_vector (i64 (bitcast (v2i32 X))), (i64 (bitcast (v2i32 X)))) -> (v2i64 (bitcast (concat_vectors (v2i32 X), (v2i32 X))))
            - "Build" Vector of ZExt from Extracting ZExt and similar combinations
            - "Convert" Building Vector
             https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L23162-L23166
            - SPLAT : Special Building Operation of broadcast. Target Action associated with it "Expand"      
      + DAGCombiner::visitINSERT_VECTOR_ELT();
          - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L531-L538
          - Cases
              + Inserting At index out of bounds for vector -> Return UNDEF
              + Remove redundant insertions (insert_vector_elt x (extract_vector_elt x idx) idx) -> x
              + Insert "Value" in "Undef" vector  -> chose to do broadcast
              + special cases : insert_vector_elt(x, extract_vector_elt(y, 0), 0) -> y
              + Canonicalize insert (insert_vector_elt (insert_vector_elt A, Idx0), Idx1) -> (insert_vector_elt (insert_vector_elt A, Idx1), Idx0)
                https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L22278-L22285
              + Merging Insert into Shuffle Vector
                https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L22049-L22055
              + Inserting SubVector
                https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L22079-L22085            
              + Insert From Load
                https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L22140-L22150
              + Using BUILD_VECTOR instead of INSERT_VECTOR
              + Using SHUFFLE_VECTOR instead of some INSERT_VECTOR
                
      + DAGCombiner::visitEXTRACT_VECTOR_ELT();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L540-L550
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L22715
        - Cases
            + Extract from UNDEF vector
            + Extract of Insert with same index  `extract_vector_elt (insert_vector_elt vec, val, idx), idx) -> val`
            + Extract of SCALAR_TO_VECTOR `(vextract (scalar_to_vector val, 0) -> val`
            + Extract from Out of bound index is UNDEF
            + Extracting from BUILD_VECTOR `extract_vector_elt (build_vector x, y), 1 -> y`
            + Extracting from VECTOR_SHUFFLE ` (EXTRACT_VECTOR_ELT( VECTOR_SHUFFLE )) -> EXTRACT_VECTOR_ELT`
            + Extracting from Load `extract (vector load $addr), i --> load $addr + i * size`
            + Extracting from CONCAT_VECTOR
                - `extract_vector_elt (concat_vectors v2i16:a, v2i16:b), 0  -> extract_vector_elt a, 0`
                - `extract_vector_elt (concat_vectors v2i16:a, v2i16:b), 1  -> extract_vector_elt a, 1`
                - `extract_vector_elt (concat_vectors v2i16:a, v2i16:b), 2 -> extract_vector_elt b, 0`
                - `extract_vector_elt (concat_vectors v2i16:a, v2i16:b), 3 -> extract_vector_elt b, 1`
                  
      + DAGCombiner::visitCONCAT_VECTORS();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L551-L558
      + DAGCombiner::visitEXTRACT_SUBVECTOR();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L575-L587
      + DAGCombiner::visitVECTOR_SHUFFLE();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L607-L615
      + DAGCombiner::visitSCALAR_TO_VECTOR();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L628-L634
      + DAGCombiner::visitINSERT_SUBVECTOR();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L559-L773
          
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
  

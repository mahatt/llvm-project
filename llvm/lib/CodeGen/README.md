
# CODE GENERATION 
* Reads llvm-ir, generates DAG
* Legalizes Type "generically"
* Legalizes operation "generically"
* Optimizes DAG with Combiner "generically"
* Scheduler
## DAG Combiner
  + Runs from
  + https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L1729-L1735

  + Node Visit/Traversal Algo
  + https://github.com/mahatt/llvm-project/blob/da7fb88f431a13aea40109bdd0e4c9f89b14011e/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L1753-L1790

  + Visiting Individual Combination Patterns

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
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L24337
        - Cases
            + All operands are UNDEF
            + all but the first of the vectors are undef
            + Concat large vector by padding UNDEF
            + ` concat_vectors(scalar_to_vector(scalar), undef) ->  scalar_to_vector(scalar)`
            + `concat_vectors(scalar, undef) -> scalar_to_vector(scalar)`
            + `fold (concat_vectors (BUILD_VECTOR A, B, ...), (BUILD_VECTOR C, D, ...)) -> (BUILD_VECTOR A, B, ..., C, D, ...)`
            + Concat Vectors of Scalars
            + `Fold CONCAT_VECTORS of CONCAT_VECTORS (or undef) to VECTOR_SHUFFLE.`
            + `Fold CONCAT_VECTORS of EXTRACT_SUBVECTOR (or undef) to VECTOR_SHUFFLE.`
            + Single Source for concat
              
      + DAGCombiner::visitEXTRACT_SUBVECTOR();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L575-L587
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L24926
        - Cases
            + extract UNDEF
            +  Combine an extract of an extract into a single extract_subvector `ext (ext X, C), 0 --> ext X, C`
            +  `ty1 extract_vector(ty2 splat(V))) -> ty1 splat(V)`
            +  `extract_subvector(insert_subvector(x,y,c1),c2)  --> extract_subvector(y,c2-c1)`
            +  `extract_subv (bitcast X), Index --> bitcast (extract_subv X, Index')`
            +  ` extract_subvec (concat V1, V2, ...), i --> Vi`
            +  `v2i8 extract_subvec (v16i8 concat (v8i8 X), (v8i8 Y), 14 -->  v2i8 extract_subvec v8i8 Y, 6`
            +  `(extract_subvec (insert_subvec V1, V2, InsIdx), ExtIdx) => (extract_subvec V1, ExtIdx)`
              
      + DAGCombiner::visitVECTOR_SHUFFLE();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L607-L615
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L25911
        - Cases
            +  Canonicalize `shuffle undef, undef -> undef`
            +  Canonicalize `shuffle v, v -> v, undef`
            +  Canonicalize `shuffle undef, v -> v, undef`
            +  Combine shuffles of splat-shuffles ` (shuffle (shuffle v, undef, SplatMask), undef, UserMask)`
                -  `UserMask=[0,2,u,u], SplatMask=[2,u,2,u] -> [2,2,u,u] `
                -  `UserMask=[3,u,2,u], SplatMask=[2,u,2,u] -> [u,u,2,u]`
             
            + Combine shuffle of shuffle `huf (shuf X, undef, InnerMask), undef, OuterMask --> splat X`
            + For Splat arg, if arg evctor is splat or build_vector
                - `splat (vector_bo L, R), Index -->splat (scalar_bo (extelt L, Index), (extelt R, Index))`
            + Shuffle of a concat of the same narrow vector  ->  low-half elements of a concat with undef
              `shuf (concat X, X), undef, Mask --> shuf (concat X, undef), undef, Mask'`
            + Replace a shuffle with an insert_subvector `shuffle(lhs,concat(rhs0,rhs1,rhs2,rhs3),0,1,2,3,10,11,6,7) -> insert_subvector(lhs,rhs1,4)`

            + Compute the combined shuffle mask for a shuffle with SV0 as the first operand, and SV1 as the second operand.
               - `Merge SVN(OtherSVN, N1) -> shuffle(SV0, SV1, Mask) iff Commute = false`
               - `Merge SVN(N1, OtherSVN) -> shuffle(SV0, SV1, Mask') iff Commute = true`
            + Asking Target Is Shuffle Mask is Legal
              https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L26397-L26405
            + Canonicalize shuffles
              - `shuffle(A, shuffle(A, B)) -> shuffle(shuffle(A,B), A)`
              - `shuffle(B, shuffle(A, B)) -> shuffle(shuffle(A,B), B)`
              - `shuffle(B, shuffle(A, Undef)) -> shuffle(shuffle(A, Undef), B)`
              - `shuffle(splat(A,u), shuffle(C,D)) -> shuffle'(shuffle(C,D), splat(A,u))`
            + Fold Shuffles
              - `shuffle(shuffle(A, B, M0), C, M1) -> shuffle(A, B, M2)`
              - `shuffle(shuffle(A, B, M0), C, M1) -> shuffle(A, C, M2)`
              - `shuffle(shuffle(A, B, M0), C, M1) -> shuffle(B, C, M2)`
            + Binary Op with Shuffle Args
              - `shuffle(bop(shuffle(x,y),shuffle(z,w)),undef)`
              - `shuffle(bop(shuffle(x,y),shuffle(z,w)),bop(shuffle(a,b),shuffle(c,d)))`
      + DAGCombiner::visitSCALAR_TO_VECTOR();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L628-L634
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L26565
        - Cases
            + Convert a scalar binop with an extracted vector element to a vector bin op
                - `s2v (bo (extelt V, Idx), C) --> shuffle (bo V, C'), {Idx, -1, -1...}`
                - `s2v (bo C, (extelt V, Idx)) --> shuffle (bo C', V), {Idx, -1, -1...}`
      + DAGCombiner::visitINSERT_SUBVECTOR();
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/include/llvm/CodeGen/ISDOpcodes.h#L559-L773
        - https://github.com/mahatt/llvm-project/blob/3384641be69120fc876cf3ac87aa9b5053812350/llvm/lib/CodeGen/SelectionDAG/DAGCombiner.cpp#L26658
        - Cases
            + Inserting Undef
            + Insert of an extracted vector into an undef vector ` insert_vec UNDEF (extract_vec V1 V2) -> use V1 V2 extract`
            + `insert_subvector(N0, extract_subvector(N0, N2), N2) --> N0`
            + `insert_subvector undef, (splat X), N2 -> splat X`
            + `INSERT_SUBVECTOR UNDEF (BITCAST N1) N2 ->BITCAST (INSERT_SUBVECTOR UNDEF N1 N2)`
            + `INSERT_SUBVECTOR (BITCAST N0) (BITCAST N1) N2 -> BITCAST (INSERT_SUBVECTOR N0 N1 N2)`
            + `INSERT_SUBVECTOR( INSERT_SUBVECTOR( Vec, SubOld, Idx ), SubNew, Idx ) -> INSERT_SUBVECTOR( Vec, SubNew, Idx )`
            + `INSERT_SUBVECTOR( INSERT_SUBVECTOR( Vec, SubOld, Idx ), SubNew, Idx ) --> INSERT_SUBVECTOR( Vec, SubNew, Idx )`
            + `insert_subvector undef, (insert_subvector undef, X, 0), 0 -->insert_subvector undef, X, 0`
            + `insert_subvector(bitcast(v), bitcast(s), c1) -> bitcast(insert_subvector(v, s, c2))`
            + Canonicalise
                - `(insert_subvector (insert_subvector A, Idx0), Idx1) -> (insert_subvector (insert_subvector A, Idx1), Idx0)`
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
  

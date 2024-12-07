# Global Instruction Selection
* Generic Classes https://github.com/mahatt/llvm-project/tree/main/llvm/lib/CodeGen/GlobalISel
* Target Specific Classes https://github.com/mahatt/llvm-project/new/main/llvm/lib/Target/X86/GISel

## Why Global ISEL?
* SelectionDAG fast code at expense of time to produce it. Building DAG is costly, does not scale since large, monolithic algo.
* FastIsel short time to generate code at expense of less optimized code. It does not create DAG.
* GISEL considers Machine Function instead of Basic Block
* GISEL uses Pass Infra.

## [Generic Machine Instruction](https://llvm.org/docs/GlobalISel/GMIR.html) (gMIR)
*  Shares same Data Structure as MachineIR(MIR) but with relaxed constraints
*  As Pipeline convert gMIR into MIR pass by pass constraints are enforced.
*  Uses Generic Opcodes starts with Prefix G_
*  Generic Virtual Registers
    + TODO
  
  
## [GISEL Core Pipeline](https://llvm.org/docs/GlobalISel/Pipeline.html)
* [IRTranslator](https://llvm.org/docs/GlobalISel/IRTranslator.html)
  + LLVM IR -> Generic Machine Instructions or MIR in Machine Basic Blocks of Machine Function
  + Translates
    - IR
    - Target Intrinsic
    - Function Calls
       + Lowering Arguments
         https://github.com/mahatt/llvm-project/blob/2c3684128e0b82c3bc7b67bc535a04c76571f0a6/llvm/lib/Target/X86/GISel/X86CallLowering.cpp#L260-L280
       + Lowering Return
         https://github.com/mahatt/llvm-project/blob/2c3684128e0b82c3bc7b67bc535a04c76571f0a6/llvm/lib/Target/X86/GISel/X86CallLowering.cpp#L144-L160
         
       + Lower Call
         https://github.com/mahatt/llvm-project/blob/2c3684128e0b82c3bc7b67bc535a04c76571f0a6/llvm/lib/Target/X86/GISel/X86CallLowering.cpp#L319
 
         * Passing Parameter
          - https://github.com/mahatt/llvm-project/blob/2bfa54fb93841124defba524fef590d393265ba4/llvm/include/llvm/CodeGen/GlobalISel/CallLowering.h#L329-L335
          - Assign Value to register, Register becomes live in Machine Basic block
          - https://github.com/mahatt/llvm-project/blob/2bfa54fb93841124defba524fef590d393265ba4/llvm/include/llvm/CodeGen/GlobalISel/CallLowering.h#L341-L348
          - https://github.com/mahatt/llvm-project/blob/2bfa54fb93841124defba524fef590d393265ba4/llvm/lib/Target/X86/GISel/X86CallLowering.cpp#L93-L120
          * Returning Parameter
            https://github.com/mahatt/llvm-project/blob/2bfa54fb93841124defba524fef590d393265ba4/llvm/include/llvm/CodeGen/GlobalISel/CallLowering.h#L345-L349
         
    - Aggregates
    - Constants
* [Generic MIR -> Legalized MIR](https://llvm.org/docs/GlobalISel/Legalizer.html)
  + Legalize all operation, beyond this there should be no illegal operation
  + LLVM IR with unsupported types and Operation -> gMIR -> Legal gMIR
  + Legality depends on Instruction itself and not on "Context"
  + Legal Instruction if Target ISEL Pattern matches, "selectable" and vregs that can be loaded and stored.
  + Legalization is iterative and stateful in GMIR. It can introduce special instructions like G_MERGE_VALUES, G_UNMERGE_VALUES and G_EXTRACT. These are expected to combine at end of legalizer, if not they are selectable.
  + LegalityQuery
    - Build `LegalizeRuleSet` using  `getActionDefinitionsBuilder`
    - https://github.com/mahatt/llvm-project/blob/75d368527d394b5a8894742ee89a6704cef62911/llvm/lib/Target/X86/GISel/X86LegalizerInfo.cpp#L137-L165
    - Working 
        + `getActionDefinitionsBuilder' builds rule-set for opcode or list of opcodes, bound permanently
        + Rules execute top to bottom, if as result of rule instruction gets legalized, then executes again from top. If does not satify any rule, it is not supported.
        + Declare Legal instruction as top level rule to avoid expensive checks.
        + Rules Action
          + `LegalIf()` `LegalFor()`
        + Rule Predicate
          + `legal()`, `lower()`
         
    - Legality of vectors, pointers, Operations
* Map Operands of MIR to Register Banks
  + All Virtual register must have register *bank*, No virtual reg should be unassigned.
* Replace Generic MIR with Real MIR by Pattern Matching
  + All gMIR will be present after pass. gMIR to MIR is Complete.
* Combiner
  + Replace pattern of instruction with optimization criteria runtime, code size etc.
  + Combiner run after IRTranslator and after Legalizer 
### Entry Point
* Adding ISEL Passes
  https://github.com/mahatt/llvm-project/blob/2c3684128e0b82c3bc7b67bc535a04c76571f0a6/llvm/include/llvm/Passes/CodeGenPassBuilder.h#L395-L400
* All passes under GISEL
  https://github.com/mahatt/llvm-project/blob/2c3684128e0b82c3bc7b67bc535a04c76571f0a6/llvm/lib/CodeGen/GlobalISel/GlobalISel.cpp#L17
* Running Generic Instruction Selection
  https://github.com/mahatt/llvm-project/blob/2c3684128e0b82c3bc7b67bc535a04c76571f0a6/llvm/lib/CodeGen/GlobalISel/InstructionSelect.cpp#L135
### References
* [LLVM GISEL DOC](https://llvm.org/docs/GlobalISel/index.html)

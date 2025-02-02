# Scheduler
* Objective for straight line code, reorder instructions to achieve
  + Hide latency
  + Increase throughtput

* LLVM Scheduling Model
  + Each Target Processor =>  Scheduler model depending on Feature Enabled
  + Machine Scheduler

    - Problem with Per Opcode Model
      + Single Opcode can have different addressing mode which have different latency
      + e,g, lat(ADD R0, R1) < lat(ADD R0, [RSI])
    - Per Operand Model
      + For Each Instruction Assign token which represent Scheduling property for Instruction or Group of Instructions
        - Step 1: Assign Instruction with SCHEDULE TOKEN
        - Step 2: Find Def and Use operands, map then with SchedWrite and SchedRead tokens, Build SCHEDULE TOKEN with SCHEDWRITE AND SCHEDREAD.
        - Step 3: Bind Processor Information with SCHEDULE TOKEN using WRITERES, denotes WRITERES represent use of Processor resource e.g port, pipeline
          Binding provides Latency, Resources used, ReleaseAtCycle (Finishing at each resource)
      + SchedRead in LLVM default does not have latency property or consume any cycle, Since All operand reads are instant. When read is not instant
        we can update it with latency e.g.
        https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/lib/Target/X86/X86Schedule.td#L24-l28
      + https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/lib/Target/X86/X86Schedule.td#L134
        https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/lib/Target/X86/X86Schedule.td#L45-L60
      + INSTRUCTION to SCHEDULE TOKEN
        https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/lib/Target/X86/X86InstrArithmetic.td#L1096-L1100
      + SCHEDULE TOKEN to Processor Resources
        https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/lib/Target/X86/X86SchedSapphireRapids.td#L129
        https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/lib/Target/X86/X86SchedSapphireRapids.td#L65-L69
      + https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/lib/Target/X86/X86SchedSapphireRapids.td#L745-L752
   
        
      + SUPER SCALAR  MICROARCHITECTURE details in SCHEDULE
        - FETCH INSTRUCTION, DECODE INSTRUCTION, DISPATCH MICRO-OPS-EXECUTE on PORTS, ROB
        - https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/include/llvm/Target/TargetSchedule.td#L132-L193
        - BufferSize maps different cases
            + BufferSize = -1 , Unitfied Reservation Station , min size of ( Register Rename, URS, ROB)
            + BufferSize = 0, ???
            + BufferSize = 1 , Inorder execution units, Inorder pipeline with Out of order core
            +  BufferSize > 1, Out of ORder execution units, Size same as Reservation station on each.
        - ProcResourceKind
            + Number of resources available to run in parallel
            + Distinction of Resources e.g. INT and FP
              
        -  ProcResourceUnits - Number of Units of a Kind
        -  ProcResource - Resource of A Type with Number of units each
        -  ProcResGroup - List of Resources with each Resource may be same or different TYPE
          https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/include/llvm/Target/TargetSchedule.td#L187-L205
          https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/lib/Target/X86/X86SchedSapphireRapids.td#L35
          https://github.com/mahatt/llvm-project/blob/7dc4d08b8cc47314923b35fcbae587589bd2ebdf/llvm/lib/Target/X86/X86SchedSapphireRapids.td#L52
              
  + Machine Combiner for target specific optimizaiton
  



# References
* https://myhsu.xyz/llvm-sched-model-1/
* https://myhsu.xyz/llvm-sched-model-1.5/
* https://github.com/mahatt/llvm-project/blob/main/llvm/include/llvm/Target/TargetSchedule.td

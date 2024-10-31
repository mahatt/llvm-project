
## Prologue 
* setup  stack frame
* callee-saved registers during the beginning of a function
* Add jump labels for exception handling frame used by eh handler.
  https://github.com/mahatt/llvm-project/blob/98815f7878c3240e27f516e331255532087f5fcb/llvm/lib/Target/X86/X86FrameLowering.cpp#L1448-L1456
  https://github.com/mahatt/llvm-project/blob/98815f7878c3240e27f516e331255532087f5fcb/llvm/lib/Target/X86/X86FrameLowering.cpp#L1533-L1550
*  Useful Data structures
  + MachineFrameInfo
  + X86MachineFunctionInfo 
  + EHPersonality
## Epilogue
* cleans up the stack frame prior to function return

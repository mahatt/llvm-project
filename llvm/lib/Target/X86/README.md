# What is Frame Index
* LLVM Codegeneration framework has concept of virtual frame throughout code generation
* Frame stores stack elements in function, numbered with indices
* Prologue allocates stack frame and Target uses virtual frame with target specific frame.
  Each Index is replaced by satck start + offset.
  https://github.com/mahatt/llvm-project/blob/bd0c5e4d04efcf1698a20b59df30563b65016d1e/llvm/lib/Target/X86/X86RegisterInfo.cpp#L890-L930

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

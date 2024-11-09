# TARGET backend

## Physical Code Organization
* CodeGen 
* MC
* TableGen
* Target
```
libLLVMAggressiveInstCombine.a
libLLVMAnalysis.a
libLLVMAsmParser.a
libLLVMAsmPrinter.a
libLLVMBinaryFormat.a
libLLVMBitReader.a
libLLVMBitWriter.a
libLLVMBitstreamReader.a
libLLVMCFGuard.a
libLLVMCFIVerify.a
libLLVMCGData.a
libLLVMCodeGen.a
libLLVMCodeGenTypes.a
libLLVMCore.a
libLLVMCoroutines.a
libLLVMCoverage.a
libLLVMDWARFLinker.a
libLLVMDWARFLinkerClassic.a
libLLVMDWARFLinkerParallel.a
libLLVMDWP.a
libLLVMDebugInfoBTF.a
libLLVMDebugInfoCodeView.a
libLLVMDebugInfoDWARF.a
libLLVMDebugInfoGSYM.a
libLLVMDebugInfoLogicalView.a
libLLVMDebugInfoMSF.a
libLLVMDebugInfoPDB.a
libLLVMDebuginfod.a
libLLVMDemangle.a
libLLVMDiff.a
libLLVMDlltoolDriver.a
libLLVMExecutionEngine.a
libLLVMExegesis.a
libLLVMExegesisX86.a
libLLVMExtensions.a
libLLVMFileCheck.a
libLLVMFrontendDriver.a
libLLVMFrontendHLSL.a
libLLVMFrontendOffloading.a
libLLVMFrontendOpenACC.a
libLLVMFrontendOpenMP.a
libLLVMFuzzMutate.a
libLLVMFuzzerCLI.a
libLLVMGlobalISel.a
libLLVMHipStdPar.a
libLLVMIRPrinter.a
libLLVMIRReader.a
libLLVMInstCombine.a
libLLVMInstrumentation.a
libLLVMInterfaceStub.a
libLLVMInterpreter.a
libLLVMJITLink.a
libLLVMLTO.a
libLLVMLibDriver.a
libLLVMLineEditor.a
libLLVMLinker.a
libLLVMMC.a
libLLVMMCA.a
libLLVMMCDisassembler.a
libLLVMMCJIT.a
libLLVMMCParser.a
libLLVMMIRParser.a
libLLVMObjCARCOpts.a
libLLVMObjCopy.a
libLLVMObject.a
libLLVMObjectYAML.a
libLLVMOptDriver.a
libLLVMOption.a
libLLVMPasses.a
libLLVMProfileData.a
libLLVMRemarks.a
libLLVMRuntimeDyld.a
libLLVMSandboxIR.a
libLLVMScalarOpts.a
libLLVMSelectionDAG.a
libLLVMSupport.a
libLLVMSymbolize.a
libLLVMTableGen.a
libLLVMTableGenBasic.a
libLLVMTableGenCommon.a
libLLVMTarget.a
libLLVMTargetParser.a
libLLVMTextAPI.a
libLLVMTextAPIBinaryReader.a
libLLVMTransformUtils.a
libLLVMVectorize.a
libLLVMWindowsDriver.a
libLLVMWindowsManifest.a
libLLVMXRay.a
libLLVMipo.a
// Parsing Assembly and Assembler
libLLVMX86AsmParser.a
// Codegeneration algorithms for X86, Specialization of libLLVMCodeGen.a
// Includes Instruction Selection,  Scheduling, Register Allocaiton
libLLVMX86CodeGen.a

// MC Level Targert Specific classes, Target Specific MC functionality 
// e.g. Code Emitter
libLLVMX86Desc.a
// Target Dependent dissassembly, Reads binary bytes and Builds MCInst Object.
libLLVMX86Disassembler.a
// Register Target with LLVM Generic Codegen in order to allow backend specific callbacks
libLLVMX86Info.a
libLLVMX86TargetMCA.a
```

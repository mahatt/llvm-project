# Clang Utility
* ABITest
  - Generates ABI test with different types
* CIndex
* CaptureCmd
  +  Captures arguments to any command in absolute PATH.
  +  Normally if  tool foo internally invokes bar as process with args, to capture args we use capturecmd.
* ClangVisualizers
* CmpDriver
  - For each tool like gcc and clang which supports -###, compare tool invocations in terms of exit codes and options added
* FindSpecRefs
* FuzzTest
  - Generic Fuzz testing tool
* TableGen
    - Clang tablegen has seperate backend for clang
* TestUtils
* VtableTest
* analyze_safe_buffer_debug_notes.py
* analyzer
* bash-autocomplete.sh
  - Useful script to autocomplete clang options on bash, for using we need to only source script.
* builtin-defines.c
* bundle_resources.py
* check_cfc
* clangdiag.py
  + clang diagnostic breakpoints
  + only works in lldb.
    
* convert_arm_neon.py
* creduce-clang-crash.py
  + Calls C-Reduce to create a minimal reproducer for clang crashes
  + TODO: Example
* find-unused-diagnostics.sh
* hmaptool
* make-ast-dump-check.sh
* modfuzz.py
* module-deps-to-rsp.py
* perf-training
* token-delta.py
* valgrind

# LLVM Utilities
1. FileCheck
2. Tablegen
3. Bugpoint
4. Docker
5. Editor Config
     + Emacs
     + vim
     + jedit
     + kate
     + textmate
     + vscode
         - TODO
     + vim
         - TODO
6. fcmp
  + Compare floating point numbers within two files with tolerance
  + Noisy compare

7. gdb-scripts
   + GDB Pretty Printing for LLVM Classes
   + It supports printing ADT like StringRef, DenseMap, SmallVector Printing.
   + Use: `(gdb) source llvm/utils/gdb-scripts/prettyprinters.py`
9. git
10. llvm-lit
11. not
    + crash testing `not llc` will return true if `llc` command did not crash
    + `not --crash llc` will return true if `llc` command crashes
12. Reducing Pipeline
    + Useful transformation in reproducing crash
13. Split File

14. Update Scripts
	+ update_analyze_test_checks.py
	+ update_any_test_checks.py
	+ update_cc_test_checks.py
	+ update_llc_test_checks.py
    - For IR file, add/update CHECK labels specified in RUN command
    - Removes old labels if already present before adding new
    - `update_llc_test_checks.py --llc-binary=<llc_binary_path> <ir_test>`
	+ update_mca_test_checks.py
	+ update_mir_test_checks.py
	+ update_test_body.py
	+ update_test_checks.py
	+ update_test_prefix.py
15. Useful Scripts
    + BISECT
      - Find guilty commit
      - `bisect --start=<start_num> --end=<end_num> ./script.sh "%(count)s"`
      - script.sh must return sucess for good commit
    + DSAclean.py
      - Remove nodes which has labels of format "%tmp.#"
    + DSAextract.py
      - Extract interested nodes from graph in dot file
    + abtest.py
    + add_argument_names.py
    + bugpoint_gisel_reducer.py
    + chunk-print-before-all.py
    + create_ladder_graph.py
    + demangle_tree.py
    + extract-section.py
    + extract_symbols.py
    + extract_vplan.py
      -  extracts the VPlan digraphs from the vectoriser into dot file
    + indirect_calls.py
    + lldbDataFormatters.py
    + llvm-gisel-cov.py
    + llvm-mca-compare.py
      - Compare two report for machine code analyser interms of scheduling parameters.
      - TODO
    + llvm-original-di-preservation.py
    + pipeline.py
    + reduce_pipeline.py
       -  for given input ir and output ir, find reduced pipeline that reproduces issue.
       - `reduce_pipeline.py --opt-binary=./build-all-Debug/bin/opt --input=input.ll --output=output.ll --passes=PIPELINE [EXTRA-OPT-ARGS ...]`
    + relative_lines.py
    + remote-exec.py
        - Run Code on remote machine
    + rsp_bisect.py
    + schedcover.py
    + sort_includes.py
    + findoptdiff
      - Find missing optimization between two llvm builds.
      - `findoptdiff llvm1 llvm2 bc1 bc2 `
    + findmisopt
      - Given IR producing correct llc output, run all passes that are applicable and has not used to identify
        missing optimization that can correctly generate optimized output.
    + findsym.pl
      - Find Definition or reference of Symbol in LLVM binary libs
      - `findsym.pl $LLVM_BUILD_LIB_LOC $SYMBOL_SEARCH_KEY`

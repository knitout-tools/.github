# knitout-tools

A collection of tools for the [knitout](https://textiles-lab.github.io/knitout/knitout.html) machine knitting language, currently maintained by [Nat Hurtig](https://nathurtig.com). Nat is not the creator or maintainer of knitout; these are unofficial. The plan, divided into library crates in order of least to most dependent then binary crates:
- tree-sitter for knitout
- Some base crate that defines the bed, needle, and direction types used throughout knitout analysis
- an ADT for parsed, but possibly invalid, knitout with spans pointing to the source characters (slightly more structure than tree-sitter's output)
- an ADT for validated knitout programs with some provenance, either a span, an "I can't be bothered to tell you where this came from" provenance, or a custom string ("generated from line XYZ of high-level DSL")
- an ADT for SWG machine passes that knows how to rasterize itself to machine binary (no more provenance). Also includes the collection of errors that might happen when rasterizing
- an ADT for parse-time errors (i.e., invalid knitout; e.g., "Wrong argument order: knit f4 + 2") that incorporates the spans
- an ADT for virtual run-time warnings (e.g., knitting on empty needle) and errors (e.g., knitting with a not-yet-in carrier) that also incorporates the spans

Relying on those:
- a parser that takes tree-sitter's semistructured output to possibly-invalid-knitout+parser-errors, and possibly-invalid-knitout to Option<validated-knitout>
- a formatter that takes possibly-invalid-knitout to formatted knitout-like-stuff where for all strings x, parse(tree-sitter(fmt(parse(tree-sitter(x))))) = parse(tree-sitter(x)) in theory
- a virtual knitting machine that executes possibly-invalid knitout up until the knitout becomes invalid, and returns runtime warnings and errors that it's sure will happen after parse errors are fixed
- a compiler that takes valid knitout to passes in the same in-knitout-order way that existing compilers do. It's unclear whether the compiler will rely on the VM's logic or whether it will be slightly duplicated. The compiler will either return the passes or it will raise one (just the first one) fatal error from the VM. The VM is for prescriptive warnings; the compiler will only stop when it's impossible to proceed.

All of the above are libraries; only tree-sitter does file I/O. The rest take/produce strings, ADTs, Vec<u8>...

Binaries:
- an LSP that uses tree-sitter=>parse=>VM, passing the parse and VM diagnostics to vscode-languageserver
- one crate to rule them all that provides "knitout fmt", "knitout check (the VM analysis)", "knitout compile (the VM analysis + attempt to compile if no errors)" that runs directly on knitout files

All of the above are currently planned to live in the same repo, but have separate crates (hence separate versioning).

After all that: possibly an algebra-based compiler that does advanced rewrites.

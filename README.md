# Reproducer project for the non-working incremental compilation with rules_swift

This project demonstrates that large `swift_library` targets never compile incrementally even though `-incremental`flag is passed.

## Steps to reproduce

1. Clone this repo
2. Install "bazelisk" `brew install bazelisk`
3. Make sure that `ac/swift-incremental-compilation-not-working` branch is checked out
4. Build the iOS app `bazel build //app`
5. Modify single source file (e.g. app/source/ContentView.swift)
6. Build again `bazel build //app`

Notice that compilation is still very slow, it is almost like everything cached is invalidated. Yes it is understandable that primary module is invalidated since it contains changes, but Swift compiler should not recompile the whole module. It is expected that only affected source files are recompiled.
Also note that it does not matter if the change modifies the public interface or not, it always recompiles whole module. This is not the case when building from vanilla Xcode.

## Why such a large module?

Usually the advice is to have wide module graph where each module is independent and reasonably small. I bet that for such projects this issue is not present. However, projects that are in process of being modularized should still play ok with Bazel, even if their module graph is not ideal.

## Things I tried to mitigate the issue

* Enabling whole module optimization by adding the flag to `swift_library` `copts = ["-whole-module-optimization"]`. It does not help at all but that is expected.
* Used `-enable-batch-mode`this gets included by default with whole module optimization, but I tried it anyway and it does not help in case of incremental compilation. This is also expected.
* Passed `-j`flag with number of cores on my machine `-j8`. This did help to speed up the build overall, but incremental build is still non-existent.
* Tried adding and removing `-incremental`from `copts`attribute, but that has no effect what so ever or I failed to notice it.
* Flipping `swift.use_global_index_store` on and off also does not help with incremental compilation.
* Used various configuration flags in `.bazelrc` for adjusting the number of jobs and worker instances but observed no change.
* Tried enabling whole module optimization for all modules except for the primary one but that also didn't have any effect on incremental compilation.


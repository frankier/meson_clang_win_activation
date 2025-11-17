These repo shows an bug or at least an unfortunate interaction between Meson, LLVM's clang-cl driver and passing linker flags containing `/link` through %LDFLAGS%.

There are a few ways to see the problem.

**#1** is in CI with Windows based on such a use of LDFLAGS found in the wild at: https://github.com/conda-forge/clang-win-activation-feedstock/blob/main/recipe/activate-clang-cl_win-64.bat . See https://github.com/frankier/meson_clang_win_activation/actions/runs/19420888176/job/55558119724

**#2** is by running `rattler-build` yourself on Windows: `rattler-build build --recipe conda.recipe/`

**#3** is by running meson manually -- which you can also do on Linux, but you will get other errors later if you don't set up cross compilation environment. Run:

```bash
CXX=clang-cl LD=lld-link LDFLAGS='/link' CXXFLAGS=-fuse-ld=lld-link meson setup build
```

The actual error occurs during Meson's sanity check when a command line with `/link /link` is passed to clang-cl, which results in `/link` being passed to `lld-link`, which it currently cannot handle.

```
 │ │ lld-link: error: could not open '/link': no such file or directory
 │ │ clang-cl: error: linker command failed with exit code 1 (use -v to see invocation)
 │ │ -----
 │ │ meson.build:1:0: ERROR: Compiler clang-cl.exe cannot compile programs.
```

# numeric-lua

A numerical computing stack for LuaJIT, built around one tensor engine.

Lua has excellent small libraries and a long history of numeric computing, but
the pieces rarely share an array model. `numeric-lua` is an attempt to build a
coherent stack instead: one core, several focused surfaces, and a single set of
benchmarks.

| project | what it is |
| --- | --- |
| [**luatorch**](https://github.com/numeric-lua/luatorch) | tensor core, ATen FFI, autograd, fusion compiler, `nn`/`optim` |
| [**numlua**](https://github.com/numeric-lua/numlua) | NumPy-shaped arrays, broadcasting, linalg, `.npy` I/O |
| [**luaplotlib**](https://github.com/numeric-lua/luaplotlib) | matplotlib-shaped SVG plotting |
| [**tensorlua**](https://github.com/numeric-lua/tensorlua) | thin Keras/TensorFlow-style training API |
| [**lua-polars**](https://github.com/numeric-lua/lua-polars) | DataFrame bindings over the Rust Polars engine |
| [**lua-notebooks**](https://github.com/numeric-lua/lua-notebooks) | native LÖVR notebooks with a 3D viewport |
| [**numeric-lua-agents**](https://github.com/numeric-lua/numeric-lua-agents) | org-only promo-window agent scheduler |

## Design

```
                    luatorch
        tensor core / ATen / autograd / fusion
             /              |              \
          numlua         tensorlua       luaplotlib
      NumPy-shaped     training API      SVG plots
         arrays        over nn/optim     on arrays
```

- **One engine.** `numlua` is a facade over luatorch tensors, not a second
  tensor implementation. `tensorlua` wraps `torch.nn`/`torch.optim`.
  `luaplotlib` plots arrays without owning a numeric core.
- **Lua conventions.** Dimensions, indices, labels, argmax, sort/topk, and
  gather/scatter are 1-based. File formats such as `.npy` remain byte
  compatible with NumPy.
- **Explicit performance.** Fusion, fused optimizers, and fast-math kernels are
  measured against eager PyTorch. Numbers are reported with their conditions,
  including losses.
- **Honest fallbacks.** Every fused path has an eager fallback; unsupported
  cases are documented, not hidden.

## Status

Early, but working. luatorch has a tensor core, autograd, `nn`/`optim`, a fusion
compiler, fused reductions, softmax/log-softmax, cross-entropy, and 261 tests.
`numlua`, `luaplotlib`, and `tensorlua` are the first organization MVPs.
`lua-polars` adds a columnar DataFrame surface: an unofficial LuaJIT binding to
the Rust Polars engine through a small C ABI, not another tensor core.
`lua-notebooks` is the first non-web notebook experiment: a LÖVR host, isolated
Lua kernels, and host-rendered 3D scene outputs.

## Licenses

BSD-3-Clause per repository. luatorch links against libtorch/PyTorch and vendors
Lua Fun under its MIT license; see the repository notices.

## Contributing

Read the organization-level [AGENTS.md](https://github.com/numeric-lua/.github/blob/main/AGENTS.md)
before changing a repository. Agent work lands on branches and draft PRs; the
coordinator reviews and merges.

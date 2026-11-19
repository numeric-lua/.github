# numeric-lua

A numerical computing stack for LuaJIT, built around one tensor engine.

[![❤ Sponsor on GitHub](https://img.shields.io/badge/%E2%9D%A4%20Sponsor-GitHub-ea4aaa?logo=githubsponsors&logoColor=white)](https://github.com/sponsors/Mateus-M-Reis)

Lua has excellent small libraries and a long history of numeric computing, but
the pieces rarely share an array model. `numeric-lua` is an attempt to build a
coherent stack instead: one core, several focused surfaces, and a single set of
benchmarks.

| project | what it is |
| --- | --- |
| [**luatorch**](https://github.com/numeric-lua/luatorch) | tensor core, ATen FFI, autograd, fusion compiler, `nn`/`optim` |
| [**numlua**](https://github.com/numeric-lua/numlua) | NumPy-shaped arrays, broadcasting, linalg, `.npy` I/O |
| [**luaplotlib**](https://github.com/numeric-lua/luaplotlib) | matplotlib-shaped SVG plotting |
| [**tensorflow-lua**](https://github.com/numeric-lua/tensorflow-lua) | native LuaJIT binding to the TensorFlow C API |
| [**tensorflow-lua-addons**](https://github.com/numeric-lua/tensorflow-lua-addons) | TensorFlow Addons parity subset for LuaJIT |
| [**lua-polars**](https://github.com/numeric-lua/lua-polars) | DataFrame bindings over the Rust Polars engine |
| [**lua-notebooks**](https://github.com/numeric-lua/lua-notebooks) | LÖVR 3D notebooks plus a native browser/remote-kernel front-end |
| [**numeric-lua-agents**](https://github.com/numeric-lua/numeric-lua-agents) | org-only promo-window agent scheduler (coordinator-only) |

## Design

```
                    luatorch
        tensor core / ATen / autograd / fusion
             /                     \
          numlua                luaplotlib
      NumPy-shaped             SVG plots
         arrays                on arrays

tensorflow-lua  ->  TensorFlow C API (independent runtime)
lua-polars      ->  Rust Polars (independent columnar engine)
```

- **One engine for the Lua numeric stack.** `numlua` is a facade over luatorch
  tensors, not a second tensor implementation. `luaplotlib` plots arrays without
  owning a numeric core. `tensorflow-lua` deliberately binds TensorFlow's own C
  runtime instead of wrapping luatorch.
- **Lua conventions.** Dimensions, indices, labels, argmax, sort/topk, and
  gather/scatter are 1-based. File formats such as `.npy` remain byte
  compatible with NumPy.
- **Explicit performance.** Fusion, fused optimizers, and fast-math kernels are
  measured against eager PyTorch. Numbers are reported with their conditions,
  including losses.
- **Honest fallbacks.** Every fused path has an eager fallback; unsupported
  cases are documented, not hidden.

## Status

Early, but working:

| project | status |
| --- | --- |
| **luatorch** | 0.10.0 alpha, CPU; tensor core, autograd, `nn`/`optim`, multi-layer RNN/LSTM, safetensors, fusion compiler, 303 tests |
| **numlua** | 0.2.0; NumPy-shaped facade, layout/npz/linalg, `.npy` and text/CSV I/O, 88 tests |
| **luaplotlib** | 0.2.0 MVP; SVG line/scatter/histogram/bar/error-bar plots, log axes, scene IR, 59 tests |
| **tensorflow-lua** | 0.15.0 alpha; TensorFlow C API graph runtime, trainable 2-D/3-D layers, 167 tests |
| **tensorflow-lua-addons** | 0.5.0; TensorFlow Addons parity subset, 16 tests |
| **lua-notebooks** | early; LÖVR desktop notebooks, remote LuaJIT gateway, browser/Canvas front-end |
| **lua-polars** | 0.1.0 eager MVP; Polars CSV/Parquet/SQL via a Rust C ABI |

`tensorflow-lua` builds, loads, and trains TensorFlow graphs from LuaJIT through
the official C API. `lua-polars` adds a columnar DataFrame surface: an
unofficial LuaJIT binding to the Rust Polars engine through a small C ABI, not
another tensor core. `lua-notebooks` now has two independent front-ends: the
native LÖVR host and a dependency-free browser app that talks to the remote
kernel over WebSocket.

## Licenses

BSD-3-Clause per repository. luatorch links against libtorch/PyTorch and vendors
Lua Fun under its MIT license; see the repository notices.

## Support

numeric-lua is free software under BSD-3-Clause. Donations are optional and
never unlock features or hold back builds.

<a href="https://github.com/sponsors/Mateus-M-Reis"><img src="https://img.shields.io/badge/Sponsor%20on-GitHub-ea4aaa?logo=githubsponsors&logoColor=white" alt="Sponsor on GitHub"></a>

You can also star the repositories, report bugs, or read
[SUPPORT.md](https://github.com/numeric-lua/.github/blob/main/SUPPORT.md) for
other ways to help.

## Contributing

Read the organization-level [AGENTS.md](https://github.com/numeric-lua/.github/blob/main/AGENTS.md)
before changing a repository. Scheduled agents work in isolated worktrees;
their tested branches may be fast-forwarded to `main` when the scheduler is
configured for that repository. The coordinator reviews all changes.

`numeric-lua-agents` and `.github` are **coordinator-only** and are not in the
agent scheduler.

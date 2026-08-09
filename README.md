# rules_vivado

[![BCR](https://img.shields.io/badge/BCR-rules_vivado-green?logo=bazel)](https://registry.bazel.build/modules/rules_vivado)
[![CI](https://github.com/hw-bzl/rules_vivado/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/hw-bzl/rules_vivado/actions/workflows/ci.yml)

Bazel rules for Xilinx Vivado FPGA synthesis, placement, routing, and
bitstream generation. HDL sources flow in through
[`rules_verilog`](https://registry.bazel.build/modules/rules_verilog)
(`VerilogInfo`) and
[`rules_vhdl`](https://registry.bazel.build/modules/rules_vhdl)
(`VhdlInfo`); the same `*_library` targets are reused for simulation
and synthesis.

The public surface is:

- **[`vivado_toolchain`](https://hw-bzl.github.io/rules_vivado/vivado_toolchain.html)** —
  wraps a shell script that sources your Vivado install. Mandatory:
  every `vivado_*` rule resolves it through Bazel toolchain
  resolution.
- **Per-phase rules** —
  [`vivado_synthesize`](https://hw-bzl.github.io/rules_vivado/vivado_synthesis.html),
  [`vivado_placement` / `vivado_routing`](https://hw-bzl.github.io/rules_vivado/vivado_implementation.html),
  [`vivado_write_bitstream`](https://hw-bzl.github.io/rules_vivado/vivado_bitstream.html),
  plus [`vivado_create_project`](https://hw-bzl.github.io/rules_vivado/vivado_project.html),
  [IP packaging](https://hw-bzl.github.io/rules_vivado/vivado_ip.html),
  and [`xsim_test`](https://hw-bzl.github.io/rules_vivado/vivado_simulation.html).
- **`vivado_flow`** — convenience macro chaining
  synthesis → opt → placement → place-opt → routing → bitstream.

Quick start, toolchain authoring, multi-version constraint gating, and
the full per-rule reference are hosted at
**<https://hw-bzl.github.io/rules_vivado/>**.

## Getting Vivado

A toolchain says how to run Vivado; it does not put Vivado on the
machine. Point one at an install you provisioned yourself, or let Bazel
provision it:
[`toolchains_vivado`](https://github.com/filmil/bazel_toolchains_vivado)
installs Vivado from an AMD/Xilinx unified installer archive and
registers the toolchain for you, so no shim script or
`register_toolchains` call is needed. See
[Toolchains](https://hw-bzl.github.io/rules_vivado/toolchains.html) for
both paths.

## License

Apache License 2.0

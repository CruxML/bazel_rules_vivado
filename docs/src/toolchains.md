# Toolchains

`rules_vivado` resolves the Xilinx Vivado install through Bazel
toolchain resolution. You declare a
[`vivado_toolchain`](./vivado_toolchain.md), wrap it in `toolchain(...)`,
and register it from `MODULE.bazel`. Every `vivado_*` rule then picks
it up automatically — there is no per-target `xilinx_env` to thread
through.

Registering a toolchain is **required**.

## Getting Vivado

A toolchain says *how* to run Vivado; it does not put Vivado on the
machine. There are two ways to close that gap.

**Bring your own install.** Author a `vivado_toolchain` whose `vivado`
attribute points at a shim that `exec`s an install you provisioned
yourself — on the host, or baked into a container image referenced from
an execution platform's `exec_properties`. This is what the rest of this
page and the [`vivado_toolchain`](./vivado_toolchain.md) docstring
describe, and it is the right answer when Vivado is already managed by
something else (a golden image, a config-management system, remote
execution workers).

**Let Bazel provision it.**
[`toolchains_vivado`](https://github.com/filmil/bazel_toolchains_vivado)
is a separate module that takes an AMD/Xilinx unified installer archive,
installs exactly the device families you ask for, and registers the
result as a `//vivado:toolchain_type` toolchain. Setup is one tag:

```python
bazel_dep(name = "rules_vivado", version = "{version}")
bazel_dep(name = "toolchains_vivado", version = "…")

vivado = use_extension("@toolchains_vivado//vivado:extensions.bzl", "vivado")
vivado.install(
    urls = ["file:///opt/archives/FPGAs_AdaptiveSoCs_Unified_SDI_2025.2_1114_2157_1.tar"],
    sha256 = "…",
    modules = ["Artix-7"],
)
```

No shim script, no `vivado_toolchain` target, no `register_toolchains`
call — the module registers the toolchain that `vivado_*` rules resolve.
AMD downloads require a login, so you supply the archive (a local file
or an internal mirror); the installation is cached outside Bazel's
output base and survives `bazel clean --expunge`. It is Linux-only and,
because Vivado is far too large to track as action inputs, the install
is referenced by absolute path — fine under the local sandbox, not
suitable for remote execution. See that module's README for the
component menu, licensing, and multi-version setup.

`rules_vivado` needs no configuration for either path: both produce an
ordinary `vivado_toolchain`.

## Implementing a toolchain

See [`vivado_toolchain`](./vivado_toolchain.md) — the rule's docstring
has the worked quickstart, attribute reference, and the `env` vs
`xilinx_env` framing.

## Network vs. node-locked licenses

`vivado_toolchain.requires_network` defaults to `True`, which is
correct for a floating/network license server
(`XILINXD_LICENSE_FILE=PORT@HOST`). It sets the `requires-network`
execution requirement on every `vivado_*` action.

Set it to `False` for license-free editions (Vivado ML Standard /
WebPACK) or for node-locked `.lic` files read from disk. Sandboxed and
remote-execution builds need network disabled to be reproducible
without the license server, so be deliberate here.

## Constraining toolchains

To run multiple Vivado versions side-by-side, gate each
`vivado_toolchain` with one of the per-version `constraint_value`s in
[`//vivado/constraints/version/BUILD.bazel`](https://github.com/hw-bzl/rules_vivado/blob/main/vivado/constraints/version/BUILD.bazel).
Each constraint corresponds to one entry in `VIVADO_VERSIONS` (defined
in
[`//vivado/private:versions.bzl`](https://github.com/hw-bzl/rules_vivado/blob/main/vivado/private/versions.bzl)).
The [`vivado_toolchain`](./vivado_toolchain.md) docstring has the
full multi-version walkthrough — `platform(...)` setup,
`register_execution_platforms`, and the `--platforms` switch.

For per-target switching without a global flag, use a wrapper rule
with `cfg = transition(...)`; see
[`tests/transition.bzl`](https://github.com/hw-bzl/rules_vivado/blob/main/tests/transition.bzl)
for a `with_vivado_version` wrapper that takes a list of targets and
pins the version for the whole group.

Constraints are the only mechanism — there is no parallel build-setting
/ flag-driven path. This keeps per-version metadata (constraints,
`exec_properties` like `container-image`) all on the platform object
where it belongs and avoids the two-sources-of-truth problem.

## Reference

See [`vivado_toolchain`](./vivado_toolchain.md) for the full attribute
set and [`VivadoToolchainInfo`](./vivado_providers.md) for the
resolved provider that downstream rules consume.

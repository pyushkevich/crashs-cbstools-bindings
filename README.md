# crashs-cbstools-bindings

Python bindings to select algorithms from [cbstools-public](https://github.com/piloubazin/cbstools-public)
(the CBS Tools Java library developed by Pierre-Louis Bazin and colleagues at the Max
Planck Institute for Human Cognitive and Brain Sciences), used by
[CRASHS](https://github.com/pyushkevich/crashs) for its CRUISE-based cortical
reconstruction pipeline.

## Why this exists

CRASHS previously reached this code through [NighRes](https://nighres.readthedocs.io/en/latest/),
which wraps cbstools-public via JCC (compiling the Java to a jar and statically
embedding a JVM into a compiled Python extension). That approach has never shipped a
prebuilt PyPI wheel and requires a JDK/JCC/Python-headers build step at install time.

Instead, this package AOT-compiles the five cbstools-public classes CRASHS needs
into a native shared library using [GraalVM Native Image](https://www.graalvm.org/latest/reference-manual/native-image/)
(no JVM at runtime), and exposes them to Python via `ctypes`. The result ships as an
ordinary platform-specific binary wheel — **no Java, JDK, or JVM is needed to install
or use this package**, only to build it from source.

This is split out from the `crashs` repo into its own package specifically because it
changes rarely (only when the underlying cbstools-public algorithms need a bugfix or a
new algorithm is added) while `crashs` itself changes often — bundling them together
meant every `crashs` iteration paid for slow, large, multi-platform compiled-wheel
builds it didn't need.

## Installation

```sh
pip install crashs-cbstools-bindings
```

## Building from source

Building from source requires a [GraalVM JDK](https://www.graalvm.org/downloads/) with
the `native-image` component (GraalVM for JDK 21+ ships this by default) on `PATH` —
only at build time, not at runtime.

```sh
git clone --recurse-submodules https://github.com/pyushkevich/crashs-cbstools-bindings
cd crashs-cbstools-bindings

# Build the native library once
bash native/scripts/build_native.sh macos-14   # or: ubuntu-latest, windows-2022

# Regular (non-editable) install picks up the native library correctly:
pip install .
```

Note: `pip install -e .` (editable install) does **not** work for this package,
because the compiled native library is only copied into the installed package
location, not back into the source tree that an editable install imports from. For
local development, build once as above, then manually stage the artifact into the
source tree before installing editable:

```sh
mkdir -p src/crashs_cbstools_bindings/_native_lib src/crashs_cbstools_bindings/_native_data
cp native/build/out/libcbstools_native.* src/crashs_cbstools_bindings/_native_lib/
cp -r native/data/topology_lut src/crashs_cbstools_bindings/_native_data/
pip install -e .
```

## API

Five functions, one per wrapped cbstools-public algorithm, taking/returning numpy
arrays (mirroring nighres's own function signatures and Fortran-order conventions):

- `topology_correction`
- `cruise_cortex_extraction`
- `levelset_to_mesh`
- `surface_inflation`
- `volumetric_layering`

See `src/crashs_cbstools_bindings/__init__.py` for exact signatures, and
`tests/test_bindings.py` for usage examples.

## License and attribution

This repository's own code (the Java `@CEntryPoint` wrapper layer and the Python
`ctypes` bindings) is MIT-licensed — see `LICENSE`. The compiled native library this
package ships is a derivative work of cbstools-public and is additionally subject to
cbstools-public's license, Creative Commons Attribution-ShareAlike 4.0 International
(CC BY-SA 4.0). See [THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md) for full
attribution details, the exact vendored commit, and what was and wasn't modified.

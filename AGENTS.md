# AGENTS.md

## Cursor Cloud specific instructions

Qiskit is a **hybrid Python + Rust SDK/library**, not a service. There is no server,
database, web app, or long-running process to start — "running" the product means
importing `qiskit` and executing quantum circuits locally (e.g. via the built-in
`StatevectorSampler`/`StatevectorEstimator` primitives and the transpiler).

### Environment layout
- A Python virtualenv lives at `.venv` (gitignored). Use it via `.venv/bin/python`,
  `.venv/bin/pip`, `.venv/bin/stestr`, `.venv/bin/black`, `.venv/bin/ruff`, etc., or
  activate it with `source .venv/bin/activate`.
- The Rust toolchain is pinned to 1.85 by `rust-toolchain.toml` (rustup auto-selects it).
- System package `python3-venv` (and `python3-dev`) are required to create the venv; they
  are already installed in the VM snapshot.

### Building / rebuilding
- The package is installed editable via `pip install -e . --no-build-isolation`, which
  compiles the Rust extension (`qiskit._accelerate`) with `setuptools-rust`.
- Editable installs build Rust in **debug** mode by default (faster to compile).
- IMPORTANT gotcha: editing Rust code under `crates/` does NOT hot-reload on the next
  `import qiskit`. You must recompile the extension after Rust changes:
  - `.venv/bin/pip install -e . --no-build-isolation` (debug), or
  - `.venv/bin/python setup.py build_rust --inplace --release` for an optimized build.
  Pure-Python edits under `qiskit/` take effect immediately (editable install).

### Testing
- Preferred (matches CI): `QISKIT_IGNORE_USER_SETTINGS=TRUE .venv/bin/stestr run [selector]`
  (see `Makefile` target `test_ci`). You can also use
  `.venv/bin/python -m unittest test.python.<module>`.
- Rust tests: `RUST_DEBUG=1 cargo test -p <crate>` (e.g. `qiskit-circuit`). Omitting
  `RUST_DEBUG=1` triggers a slow release build.
- `tox` is not installed in this venv; run tools directly (as above) rather than `tox -e ...`.
  Standard command mappings live in `Makefile` and `tox.ini`.

### Linting
- Python: `.venv/bin/black --check qiskit test tools setup.py docs/conf.py` and
  `.venv/bin/ruff check qiskit test tools setup.py`. Note `ruff` is pinned to the old
  `0.0.267`; always use the venv copy so results match CI.
- Rust: `cargo fmt --check` and `cargo clippy -- -D warnings`.

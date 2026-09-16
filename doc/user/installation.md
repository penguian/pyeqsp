# Appendix B: Installation & Requirements

**PyEQSP: Python Equal Area Sphere Partitioning Library**

PyEQSP requires Python 3.11 or later. We recommend using a virtual environment to manage dependencies locally.

## Prerequisites

- Python 3.11 or later
- `pip` (Python package installer)

Core library dependencies (installed automatically via `pip`):
- `numpy`
- `scipy`
- `matplotlib`

Optional dependencies:
- `pyvista` (for 3D interactive visualizations; install via
  `pip install 'pyeqsp[pyvista]'`)

## Virtual Environment Setup

Using a virtual environment prevents version conflicts between your scientific projects.
In the commands below, `.venvs/.venv` and `.venvs/.venv_sys` are the project's conventional paths
(see `INSTALL.md` in the repository root for background on virtual environments;
you may use any path that suits your setup).

Two virtual environment strategies are supported:

| Strategy | Command | Typical Use Case |
|:---|:---|:---|
| **`VENV`** | `python3 -m venv .venvs/.venv` | Standard isolated environment; installs dependencies via PyPI wheels |
| **`VENV_SYS`** | `python3 -m venv --system-site-packages .venvs/.venv_sys` | System-integrated environment; uses system packages (e.g., system VTK) |

To create and activate a standard virtual environment (`VENV`):

```bash
# Create a hidden environment directory (project convention)
python3 -m venv .venvs/.venv

# Activate it
source .venvs/.venv/bin/activate
```

To create and activate a system-site-packages virtual environment (`VENV_SYS`):

```bash
# Create a system-integrated environment (project convention)
python3 -m venv --system-site-packages .venvs/.venv_sys

# Activate it
source .venvs/.venv_sys/bin/activate
```

> [!NOTE]
> `.venvs/.venv` and `.venvs/.venv_sys` are the project's conventional virtual environment paths (where `VENV` and `VENV_SYS` denote the strategy names). Replace them with your preferred location if you are using a different layout.

## 1. Installation from Source (Git Clone)

If you have cloned the repository or are developing with the source code, install directly from the local repository directory:

```bash
# Ensure your virtual environment is active
source .venvs/.venv/bin/activate

# Inside the cloned repository root:
pip install .
```

To install with PyVista 3D visualization support:

```bash
pip install ".[pyvista]"
```

### Editable / Development Mode (Recommended for Developers)

If you intend to modify the code or run test suites and want changes to take effect immediately without reinstalling:

```bash
pip install -e .
```

To also install development and testing tools (`ruff`, `pylint`, `pytest`, `coverage`):

```bash
pip install -e ".[dev]"
```

Or with both development tools and PyVista:

```bash
pip install -e ".[dev,pyvista]"
```

## 2. Installation via Pip (PyPI)

If you are not using a Git clone and want to install **PyEQSP** as a package from PyPI:

Because the current PyPI releases are beta pre-releases, you must specify the `--pre` flag:

```bash
pip install --pre pyeqsp
```

To install with development and testing dependencies:

```bash
pip install --pre "pyeqsp[dev]"
```

To install with PyVista support:

```bash
pip install --pre "pyeqsp[pyvista]"
```

(venv-sys-setup)=
## 3D Plotting & Visualizations Setup

While 2D illustrations work with standard Matplotlib, **3D interactive visualizations** require **PyVista**.

### 1. Install PyVista

#### Standard `VENV` Path (Wheel-based VTK)
In a standard `VENV` environment, install with the `pyvista` extra:
- From local Git clone: `pip install ".[pyvista]"` (or `pip install -e ".[dev,pyvista]"`)
- From PyPI: `pip install --pre "pyeqsp[pyvista]"`

#### Using System VTK (`VENV_SYS` Path)

If you are using a `VENV_SYS` environment (required on ARM64 / Fedora Asahi Remix; optional on x86-64), install the system `python3-vtk` package first (see `INSTALL.md` in the repository root), then install PyVista and its Python dependencies without bundled VTK:

```bash
pip install --no-deps pyvista
pip install pyvista-validation scooby pillow pooch cyclopts
pip install -e ".[dev]"
```

This uses the system VTK rather than downloading the PyPI wheel.

### 2. Display Calibration & Off-Screen Rendering

PyVista supports both interactive GUI windows and headless off-screen rendering for CI
environments or Jupyter notebooks.

Off-screen rendering is controlled in Python by setting:

```python
import pyvista as pv
pv.OFF_SCREEN = True
```

> [!NOTE]
> `eqsp.visualizations` does not set `pv.OFF_SCREEN` automatically; set it in your
> script (e.g. `pv.OFF_SCREEN = True`) before calling 3D functions. The helper script
> `tests/src/inspect_visualizations.py` also honors `PYVISTA_OFF_SCREEN` by setting
> `pv.OFF_SCREEN` before running.

## Jupyter Notebook Integration

PyVista integrates with Jupyter Notebooks for interactive 3D rendering:

```bash
pip install trame ipywidgets
```

## Verification and Troubleshooting

### Verification Scripts

If you have cloned the repository, verification scripts are provided to inspect visual outputs:

- **2D Illustrations (Matplotlib)**:
  ```bash
  python tests/src/inspect_illustrations.py
  ```
- **3D Visualizations (PyVista)**:
  ```bash
  python tests/src/inspect_visualizations.py
  ```

### Troubleshooting PyVista

If PyVista fails to open a window:
1. Verify `echo $DISPLAY` is set, or enable off-screen mode via `import pyvista as pv; pv.OFF_SCREEN = True` in your script.
2. Run `python tests/src/inspect_visualizations.py` to verify PyVista rendering.
3. **ARM64 segfault on PyVista import**: The PyPI `vtk` wheel is built for 4 KB page alignment and is incompatible with ARM64 / Fedora Asahi Remix (which requires 16 KB page alignment). Use the `VENV_SYS` install path described in `INSTALL.md`.
4. **Interactive window fails on Wayland**: Try setting `QT_QPA_PLATFORM=wayland` or `QT_QPA_PLATFORM=xcb` (XWayland fallback). Not needed for automated testing or headless scripts (`pv.OFF_SCREEN = True`).

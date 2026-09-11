# QuadForge Engine

The standalone command-line remeshing engine used by the QuadForge Blender add-on. It converts triangle meshes into quad-dominant meshes and runs locally without Blender, a graphical interface, or a server.

This repository distributes the engine executables as a separately downloaded dependency of QuadForge and other products authorized by Blender Ustad. Authorized users can also invoke the engine through scripts and automated 3D workflows supporting those products. The executable is named `autoremesher` and identifies itself as AutoRemesher.

## Features

- Automatic quad-dominant remeshing with a configurable target quad count.
- Curvature-adaptive mesh density and sharp-edge controls.
- Preservation of open boundaries, with optional hole filling.
- Optional smoothing of the remeshed output.
- Symmetry across the X, Y, or Z origin plane.
- OBJ file input and output, plus streaming through standard input and output.
- Statistics reporting for quad count, non-quad count, vertices, and processing time.

## Available binaries

| Platform | Executable |
| --- | --- |
| Windows x64 | `autoremesher.exe` |
| macOS x86_64 (Intel) | `autoremesher` |

The macOS executable is an Intel build; Apple Silicon Macs require Rosetta 2. A Linux binary is not included.

## Quick start

Download or clone this repository and open a terminal in the folder containing the executable. Prepare a triangulated OBJ mesh as your input.

### Windows PowerShell

```powershell
.\autoremesher.exe --input input.obj --output output.obj --target-quads 10000
```

### macOS

Make the binary executable, then run it:

```bash
chmod +x ./autoremesher
./autoremesher --input input.obj --output output.obj --target-quads 10000
```

The result is written to `output.obj`. Use a different output path to keep your source mesh.

## Command-line options

```text
autoremesher --input <file.obj> --output <file.obj> [options]
```

| Option | Description | Default |
| --- | --- | --- |
| `-i`, `--input <path>` | Input triangle mesh in OBJ format; use `-` for stdin. | Required |
| `-o`, `--output <path>` | Output OBJ path; use `-` for stdout. | Required |
| `--target-quads <n>` | Approximate target quad count. | `20000` |
| `--edge-scaling <f>` | Edge scaling factor, from `1.0` to `4.0`. | `1.0` |
| `--sharp-edge <deg>` | Sharp-edge dihedral threshold, from `30.0` to `180.0` degrees. | `90.0` |
| `--smooth-normal <deg>` | Normal-smoothing angle threshold, from `0.0` to `180.0` degrees. | `0.0` |
| `--adaptivity <v>` | Curvature-adaptive density: `0.0` for uniform, `1.0` for fully adaptive. | `1.0` |
| `--fill-holes` | Cap open boundaries. | Off; preserve holes |
| `--smooth-iterations <n>` | Number of output smoothing passes. | `0` (off) |
| `--symmetry <axis>` | Remesh one half and mirror across the selected origin plane: `x`, `y`, `z`, or `none`. | `none` |
| `--report <path>` | Write processing statistics to a text file. | No report file |
| `-h`, `--help` | Display usage and available options. | |
| `-v`, `--version` | Display the engine version. | |

For symmetry, center the input mesh on the selected origin plane.

### Remesh with symmetry and smoothing

```powershell
.\autoremesher.exe --input input.obj --output output.obj --target-quads 10000 --symmetry x --smooth-iterations 3 --report report.txt
```

On macOS, replace `.\autoremesher.exe` with `./autoremesher`.

## Streaming and integration

Use `--input - --output -` to exchange OBJ data through pipes without temporary mesh files. In this mode, stdout contains OBJ data; progress and statistics go to stderr.

Example using Python's standard library:

```python
from pathlib import Path
import platform
import subprocess

binary = "autoremesher.exe" if platform.system() == "Windows" else "autoremesher"
engine = Path(binary).resolve()
input_obj = Path("input.obj").read_bytes()

result = subprocess.run(
    [str(engine), "--input", "-", "--output", "-", "--target-quads", "10000"],
    input=input_obj,
    capture_output=True,
)

if result.returncode != 0:
    raise RuntimeError(result.stderr.decode("utf-8", errors="replace"))

Path("output.obj").write_bytes(result.stdout)
print(result.stderr.decode("utf-8", errors="replace"))
```

Applications can supply OBJ bytes directly from memory and consume the returned OBJ bytes without reading or writing mesh files.

## Scope and limitations

The output is quad-dominant: some non-quad faces may remain, and the final quad count may differ from the requested target. Results depend on the input geometry and remeshing settings.

The engine processes mesh geometry. PBR texture baking, LOD orchestration, armature weight transfer, and shape key transfer are features of the QuadForge Blender add-on and are not provided by these executables.

## Credits

QuadForge is maintained by **Blender Ustad**. The engine is based on **AutoRemesher by Jeremy HU / dust3d**.

The AutoRemesher engine source carries an MIT license notice. Third-party components retain their respective licenses and copyright notices.

## License

The proprietary contributions are covered by the [QuadForge Engine Binary License](LICENSE.txt). It permits automatic or manual installation as a dependency of an authorized product, including use for commercial work. Generated output carries no engine royalties or attribution requirement.

Third-party components remain governed by their own licenses. The proprietary license does not override their notices, source-access requirements, or other rights.

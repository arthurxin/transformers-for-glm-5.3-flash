# Reproducing the environment

Target venv: `python3.12` (must match — `tokenizers` ships a compiled `abi3` `.so`).

## 1. transformers (this repo)

The source image installs transformers **editable** from
`/sgl-workspace/transformers`, so there is no importable copy in
`site-packages` — only a `.pth` pointing at `src/`. Reproduce that:

```bash
git clone git@github.com:arthurxin/transformers-for-glm-5.3-flash.git
pip install --no-deps --no-build-isolation -e ./transformers-for-glm-5.3-flash
```

Non-editable also works if you prefer:

```bash
pip install --no-deps ./transformers-for-glm-5.3-flash
```

## 2. tokenizers 0.23.1

Published on PyPI — no need to copy binaries out of the image:

```bash
pip install --no-deps "tokenizers>=0.23.1,<0.24.0"
```

The image's wheel is `manylinux2014` + `abi3` (needs glibc >= 2.17; Ubuntu 24.04
has 2.39), so the PyPI wheel and the in-image `.so` are interchangeable on x86_64.

## 3. torch stack — must come from the cu130 index

```bash
pip install --index-url https://download.pytorch.org/whl/cu130 \
  torch==2.13.0 torchvision==0.28.0 torchaudio==2.11.0
```

## 4. Full pinned list

`requirements-frozen.txt` is the complete 314-package manifest read out of the
image. Note some entries are not installable from public indexes:

| package | version | note |
|---|---|---|
| `sglang` | `0.0.0.dev1+gf609d677b` | **public** commit `f609d677b909` — installable, see below |
| `sglang-kernel` | `0.4.6.post1` | on PyPI |
| `sglang-router` | `0.3.2` | on PyPI |
| `sgl-deep-ep` | `0.1.2` | from `docs.sglang.ai/whl/cu129` |
| `mscclpp` | `0.9.1` | compiled in-image with custom `CMAKE_ARGS` |
| `nixl` | `1.4.0` | `nixl-cu13` variant |

### sglang is NOT patched

Unlike transformers, the sglang in this image comes from the **public** repo.
The image builds it via `COPY sglang-src/` (a CI convenience so the build never
needs network), but the version string encodes a real public commit:

```
sglang==0.0.0.dev1+gf609d677b  ->  sgl-project/sglang @ f609d677b909
  "fix(glm5_next): drop dead declare_load_time_override import"  (2026-08-25)
```

That tree already contains `python/sglang/srt/models/glm5_next.py` and
`glm5_next_nextn.py`, so no fork is needed:

```bash
pip install --no-deps \
  "git+https://github.com/sgl-project/sglang.git@f609d677b909#subdirectory=python"
```

transformers is the **only** package in the manifest that cannot be obtained
from a public index — GLM-5-Next model definitions were not upstreamed to
HuggingFace, which is why this repo exists.

## Environment versions (from image config)

```
CUDA           13.0.3
cuDNN          9.14.0.64
NCCL           2.28.3  (pip: nvidia-nccl-cu13==2.30.7)
flashinfer     0.6.17
triton         3.7.1
tilelang       0.1.12
numpy          2.3.5
python         3.12
```

## Source image

```
lmsysorg/sglang:glm-5.3-flash
sha256:d246b29aa6543ab28859439d4143bbf553b44cc74b29a98f4ee8768b401de252
```

See `PROVENANCE.md` for how this tree was extracted.

# Provenance

This tree is a **patched fork of HuggingFace transformers** extracted from a
public Docker image, for the purpose of reproducing a working GLM-5.3 / GLM-5-Next
inference environment.

## Source

| | |
|---|---|
| Docker image | `lmsysorg/sglang:glm-5.3-flash` |
| Image digest | `sha256:d246b29aa6543ab28859439d4143bbf553b44cc74b29a98f4ee8768b401de252` (linux/amd64) |
| Path in image | `/sgl-workspace/transformers` |
| `transformers.__version__` | `5.16.0.dev0` |
| Install form | editable (`pip install --no-deps --no-build-isolation -e`) |

The image installs this tree as an editable package, so
`site-packages/` contains only
`__editable__.transformers-5.16.0.dev0.pth` pointing at
`/sgl-workspace/transformers/src` — the real code lives here.

## Upstream commit

Not publicly recoverable. In the image, `.git` was a worktree pointer:

```
gitdir: /home/ubuntu/github_projects/sglang_worksapce/glmnext/transformers-glm5-next-patched/.git/worktrees/tf-pinned-0825
```

i.e. a private working tree (branch `tf-pinned-0825`) on the image builder's
machine. No public commit hash corresponds to this snapshot, which is why the
code is mirrored here rather than referenced as a git dependency.

## What's patched

Model directories not present in upstream transformers 5.x, notably:

- `src/transformers/models/glm5_next/` — configuration, modeling, processing,
  image/video processors for GLM-5-Next

## Environment it was built for

```
torch            2.13.0        (cu130 index)
torchvision      0.28.0
torchaudio       2.11.0
tokenizers       0.23.1
CUDA             13.0.3
cuDNN            9.14.0.64
NCCL             2.28.3
flashinfer       0.6.17
python           3.12
```

## Notes

- `src/transformers/testing_utils.py:219` contains a HuggingFace CI test token.
  This is inherited from upstream transformers and is not a private credential.
- License: Apache-2.0, unchanged from upstream (see `LICENSE`).

# KeyState-aware VLA

Private project repository for KeyState-aware Adaptive Action Chunking for Vision-Language-Action models.

## Repository layout

- `Idea/`: project notes and design documents.
- `third_party/RoboTwin/`: RoboTwin dependency, intended to be added as a Git submodule after the private RoboTwin mirror/fork is created.

## Clone with submodules

After the GitHub private repositories are created and RoboTwin is added as a submodule, clone with:

```bash
git clone --recurse-submodules <main-repo-url>
```

For an existing clone:

```bash
git submodule update --init --recursive
```

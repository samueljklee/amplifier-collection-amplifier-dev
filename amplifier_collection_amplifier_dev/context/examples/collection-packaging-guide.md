# Collection Packaging Guide

> Essential knowledge for packaging Amplifier collections for pip/uv installation

## The Problem

When creating a collection, a common mistake is placing collection resources (`agents/`, `context/`, `profiles/`) at the **repository root level** instead of inside the **package directory**. This causes installation failures when using:

```bash
amplifier collection add git+https://github.com/user/my-collection
```

The installation may complete without obvious errors, but the collection won't work because the resources aren't discoverable after pip/uv installation.

## The Solution: Correct Directory Structure

According to the [Collection Format Specification](https://github.com/microsoft/amplifier-collections/blob/main/docs/SPECIFICATION.md), collections must follow this structure:

```
my-collection/                          # Git repository root
  pyproject.toml                        # Build configuration (at root)
  MANIFEST.in                           # Data file inclusion rules
  README.md                             # Collection documentation
  
  my_collection/                        # Package directory (hyphens → underscores!)
    __init__.py                         # Python package marker
    pyproject.toml                      # Copy for runtime metadata discovery
    
    profiles/                           # ✓ Resources INSIDE package
      my-profile.md
    agents/                             # ✓ Resources INSIDE package
      my-agent.md
    context/                            # ✓ Resources INSIDE package
      expertise.md
```

### Key Points

1. **Package directory uses underscores**: `amplifier-collection-amplifier-dev` → `amplifier_collection_amplifier_dev/`
2. **Resources go inside package**: All `profiles/`, `agents/`, `context/`, `modules/`, `scenario-tools/` directories must be inside the package directory
3. **Two pyproject.toml files**: One at root for build config, one inside package for runtime metadata discovery
4. **MANIFEST.in includes package resources**: Must specify the package directory path

## Why This Structure?

When you install a collection via `uv pip install` or `amplifier collection add git+...`:

1. pip/uv reads the **root** `pyproject.toml` to understand how to build the package
2. pip/uv installs the **package directory** to `~/.amplifier/collections/<collection-name>/`
3. Amplifier looks for resources **relative to the installed package location**

If resources are at the root level, they won't be included in the pip installation, and Amplifier won't find them.

## Required Files

### 1. Root pyproject.toml

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "amplifier-collection-my-collection"
version = "1.0.0"
description = "My collection description"
readme = "README.md"
requires-python = ">=3.11"
license = "MIT"
authors = [
    {name = "Your Name"}
]

[tool.setuptools]
packages = {find = {}}

[tool.setuptools.package-data]
amplifier_collection_my_collection = ["*.toml", "**/*.md"]

[tool.amplifier.collection]
author = "Your Name"
capabilities = [
    "capability-1",
    "capability-2"
]
dependencies = [
    "foundation",
    "developer-expertise"
]
```

**Critical**: The `[tool.setuptools.package-data]` section must include:
- `"*.toml"` - To include the nested pyproject.toml
- `"**/*.md"` - To include all markdown files (profiles, agents, context)

### 2. MANIFEST.in

```
# Include metadata files
include README.md
include pyproject.toml

# Include all collection data inside package
recursive-include amplifier_collection_my_collection *.md
recursive-include amplifier_collection_my_collection *.toml
```

**Critical**: Use `recursive-include` with the **package directory name**, not just the resource directories.

### 3. Package pyproject.toml

Copy the root `pyproject.toml` into the package directory:

```bash
cp pyproject.toml amplifier_collection_my_collection/
```

This allows Amplifier to discover metadata after installation.

### 4. Package __init__.py

```python
"""My Collection - Description."""
__version__ = "1.0.0"
```

## Migration Steps

If you have an existing collection with resources at the root level:

```bash
# 1. Move resources into package directory
mv agents/ amplifier_collection_my_collection/
mv context/ amplifier_collection_my_collection/
mv profiles/ amplifier_collection_my_collection/

# 2. Copy pyproject.toml into package
cp pyproject.toml amplifier_collection_my_collection/

# 3. Update MANIFEST.in
# Change:
#   recursive-include agents *.md
# To:
#   recursive-include amplifier_collection_my_collection *.md

# 4. Update pyproject.toml package-data
# Add "**/*.md" to the package-data list

# 5. Git add and commit
git add -A
git commit -m "Fix collection packaging structure for pip/uv compatibility"
git push
```

## Verification

After fixing the structure, verify the collection installs correctly:

```bash
# Remove old broken installation
amplifier collection remove my-collection

# Install from GitHub
amplifier collection add git+https://github.com/user/my-collection

# List collections to verify it's installed
amplifier collection list

# Try using a profile from the collection
amplifier run --profile my-collection:my-profile --mode chat
```

## Common Mistakes

### ❌ Resources at root level
```
my-collection/
  pyproject.toml
  agents/           # Wrong! Won't be installed by pip
  profiles/         # Wrong! Won't be installed by pip
  context/          # Wrong! Won't be installed by pip
  my_collection/
    __init__.py
```

### ❌ MANIFEST.in without package directory
```
recursive-include agents *.md      # Wrong! Misses package directory
recursive-include profiles *.md    # Wrong! Misses package directory
```

### ❌ Missing package-data
```toml
[tool.setuptools.package-data]
my_collection = ["*.toml"]  # Wrong! Missing "**/*.md"
```

### ❌ Forgetting nested pyproject.toml
```
my_collection/
  __init__.py
  profiles/
  # Missing: pyproject.toml copy for runtime metadata
```

### ✅ Correct Structure
```
my-collection/
  pyproject.toml                    # Build config
  MANIFEST.in                       # Includes package directory
  my_collection/
    __init__.py
    pyproject.toml                  # Runtime metadata
    profiles/                       # Inside package
    agents/                         # Inside package
    context/                        # Inside package
```

## References

- [Collection Format Specification](https://github.com/microsoft/amplifier-collections/blob/main/docs/SPECIFICATION.md) - Section "Package Structure for Installation" (lines 53-79, 351-428)
- [Collection Authoring Guide](https://github.com/microsoft/amplifier-collections/blob/main/docs/AUTHORING.md)
- [Python Packaging User Guide](https://packaging.python.org/en/latest/tutorials/packaging-projects/)

## Real-World Example

This collection (`amplifier-collection-amplifier-dev`) was fixed using these exact steps. See commits:
- Initial fix: `dfd80eb - Fix collection packaging structure for pip/uv compatibility`
- README updates: `07121f6 - Fix README inaccuracies and collection name references`

The fix involved:
1. Moving `agents/`, `context/`, `profiles/` into `amplifier_collection_amplifier_dev/`
2. Copying `pyproject.toml` into the package
3. Updating `MANIFEST.in` to use `recursive-include amplifier_collection_amplifier_dev`
4. Adding `"**/*.md"` to `package-data` in both pyproject.toml files

After these changes, the collection successfully installs via:
```bash
amplifier collection add git+https://github.com/samueljklee/amplifier-collection-amplifier-dev
```

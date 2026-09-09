# s2i-nodejs-container Project Guide

## Overview

This repository contains the source for building various versions of Node.js container images using source-to-image (s2i). It supports multiple distributions (RHEL, CentOS Stream, Fedora) and uses distgen for template-based multi-distro generation.

## Repository Structure

```
.
├── specs/
│   └── multispec.yml          # Version and distro configuration
├── manifest.yml               # File generation rules for standard images
├── manifest-minimal.yml       # File generation rules for minimal images
├── src/                       # Jinja2 template files
│   ├── Dockerfile             # Main Dockerfile template
│   ├── Dockerfile.minimal     # Minimal Dockerfile template
│   ├── README.md              # README template
│   └── s2i/                   # S2I script templates
├── <version>/                 # Generated version directories (e.g., 22/, 24/, 26/)
│   ├── Dockerfile.rhel8
│   ├── Dockerfile.rhel9
│   ├── Dockerfile.rhel10
│   ├── Dockerfile.c9s
│   ├── Dockerfile.c10s
│   ├── Dockerfile.fedora
│   ├── README.md
│   └── ...
├── <version>-minimal/         # Generated minimal variant directories
└── common/                    # Git submodule with shared scripts
```

## Distgen Workflow

This project uses **distgen** to generate Dockerfiles and other files from templates:

1. **Templates** in `src/` use Jinja2 syntax with variables like `{{ spec.* }}`
2. **Configuration** in `specs/multispec.yml` defines:
   - `distroinfo`: Configuration for each distribution (rhel8, rhel9, rhel10, c9s, c10s, fedora)
   - `version`: Configuration for each Node.js version
   - `matrix`: Which versions are built for which distros
3. **Manifest files** (`manifest.yml`, `manifest-minimal.yml`) define rules for generating/copying files
4. **Generation** creates version-specific directories with Dockerfiles for each distro

### Key Distgen Concepts

- **Modern distros** (RHEL10, C10S, Fedora): Use versioned packages like `nodejs$NODEJS_VERSION`
- **Legacy distros** (RHEL8/9, C9S): Use DNF modules like `dnf module enable nodejs:$VERSION`
- **Minimal variants**: Use `microdnf` and minimal base images for smaller footprint
- **Template variables**: Accessed via `{{ spec.variable_name }}` in templates
- **Conditionals**: Use `{% if spec.prod == 'fedora' %}...{% endif %}` for distro-specific logic

## Available Skills

### `/add-distgen-version`

Add a new Node.js version to this repository. This skill guides you through:

1. Updating `specs/multispec.yml` with new version definition
2. Adding version to the distribution matrix
3. Generating Dockerfiles for all supported distros
4. Running verification and tests

**Usage:** Invoke when adding a new Node.js version (e.g., Node.js 26, 28, etc.)

**Example:**
```
User: "Add Node.js 26 support"
Assistant: [Invokes /add-distgen-version skill]
```

## Common Tasks

### Adding a New Version

Use the `/add-distgen-version` skill to guide you through the process.

### Regenerating Files

After modifying templates or configuration:

```bash
# Generate files for a specific version
make distgen VERSION=26

# Or generate all versions
make distgen
```

### Testing

```bash
# Build a specific version for a specific distro
make build TARGET=c9s VERSIONS=26

# Test a specific version
make test TARGET=c9s VERSIONS=26

# Test all versions
make test
```

### Verification

```bash
# Run distgen integration verification
./verify-distgen-integration.sh
```

## Important Patterns

### Version Numbering
- Standard versions: "22", "24", "26"
- Minimal versions: "22-minimal", "24-minimal", "26-minimal"

### Supported Distributions
- **RHEL 8** (`rhel8`): Legacy, uses DNF modules
- **RHEL 9** (`rhel9`): Legacy, uses DNF modules  
- **RHEL 10** (`rhel10`): Modern, uses versioned packages
- **CentOS Stream 9** (`c9s`): Legacy, uses DNF modules
- **CentOS Stream 10** (`c10s`): Modern, uses versioned packages
- **Fedora** (`fedora`): Modern, uses versioned packages

### File Generation Rules

From `manifest.yml`:
- `DISTGEN_MULTI_RULES`: Files generated for each (version, distro) combination
- `DISTGEN_RULES`: Files generated once per version
- `COPY_RULES`: Files copied without template processing
- `SYMLINK_RULES`: Symbolic links created

## Git Workflow

- **Main branch**: `master`
- **Current branch**: `add-nodejs-26` (adding Node.js 26 support)
- **Feature branches**: Create for new versions or features

## Dependencies

- **distgen**: Template generation tool (`pip install distgen`)
- **common submodule**: Shared scripts and tools (`git submodule update --init`)
- **make**: Build automation
- **podman/docker**: Container runtime for testing

## References

- [Source-to-Image (S2I)](https://github.com/openshift/source-to-image)
- [sclorg Community](https://github.com/sclorg)
- [Distgen Documentation](https://github.com/devexp-db/distgen)
- [PostgreSQL Container Example](https://github.com/sclorg/postgresql-container)

## Notes

- Never run `git push` - this is configured in memory preferences
- Always test on at least one distro before generating all variants
- Check upstream package availability before adding new distros to matrix
- Verify templates before running full distgen to catch errors early

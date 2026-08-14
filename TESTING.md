# Testing Distgen Integration

## Prerequisites

1. **Initialize git submodules:**
   ```bash
   git submodule update --init
   ```

2. **Install distgen:**
   ```bash
   # On Fedora/RHEL
   sudo dnf install distgen
   
   # Or via pip
   pip install distgen
   ```

## Regenerate Files (Verification)

To verify the distgen setup works correctly, regenerate all files and check for changes:

```bash
# Regenerate for a specific version and distro
distgen -c specs/multispec.yml --multispec version=22,distro=fedora-43-x86_64 \
        --template src/Dockerfile \
        --distro fedora-43-x86_64 \
        --multispec-selector version=22

# Or use the common makefile targets (once submodules are initialized)
make generate
```

## Build and Test Images

### 1. Build a Single Version/Target

```bash
# Build Node.js 22 for Fedora
make build TARGET=fedora VERSIONS=22

# Build Node.js 24 for CentOS Stream 9
make build TARGET=c9s VERSIONS=24

# Build minimal variant
make build TARGET=rhel9 VERSIONS=24-minimal
```

### 2. Run Tests

```bash
# Test a specific version on a specific target
make test TARGET=fedora VERSIONS=22

# Test multiple versions
make test TARGET=c9s VERSIONS="22 24"

# Run upstream tests
make test-upstream TARGET=fedora VERSIONS=22
```

### 3. Verify Generated Dockerfiles

Compare generated files with templates to ensure distgen is working:

```bash
# Check that Fedora 22 uses versioned packages
grep "nodejs\$NODEJS_VERSION" 22/Dockerfile.fedora

# Check that RHEL8 uses gcc-toolset-13
grep "gcc-toolset-13" 22/Dockerfile.rhel8

# Check that C10S is present for version 22
ls 22/Dockerfile.c10s

# Verify C8S was removed for version 22
ls 22/Dockerfile.c8s 2>&1 | grep "No such file"
```

## Manual Testing

### Test CNB Variables

```bash
# Build and inspect environment variables
podman build -t test-nodejs-22 -f 22/Dockerfile.fedora 22/
podman run --rm test-nodejs-22 env | grep CNB
# Should show:
# CNB_STACK_ID=com.redhat.stacks.fedora-nodejs-22
# CNB_USER_ID=1001
# CNB_GROUP_ID=0
```

### Test S2I Build

```bash
# Test with a sample Node.js app
s2i build https://github.com/sclorg/s2i-nodejs-container.git \
         --context-dir=22/test/test-app \
         quay.io/fedora/nodejs-22 \
         test-nodejs-app

# Run the built app
podman run -p 8080:8080 test-nodejs-app
```

## Matrix Coverage Verification

Verify all expected distro/version combinations exist:

```bash
# Node.js 20: should have rhel8, rhel9, c9s, fedora
ls 20/Dockerfile.* | wc -l  # Should be 4

# Node.js 22: should have rhel8, rhel9, rhel10, c9s, c10s, fedora
ls 22/Dockerfile.* | wc -l  # Should be 6

# Node.js 24: should have rhel8, rhel9, rhel10, c9s, c10s, fedora
ls 24/Dockerfile.* | wc -l  # Should be 6

# Verify NO c8s for v22 and v24
! ls 22/Dockerfile.c8s 2>/dev/null
! ls 24/Dockerfile.c8s 2>/dev/null
```

## Specific Tests for Distgen Features

### 1. Test Versioned Links (Fedora, RHEL10, C10S)

```bash
# Build Fedora image
podman build -t test-fedora-nodejs 22/Dockerfile.fedora 22/

# Check symlinks
podman run --rm test-fedora-nodejs ls -la /usr/bin/node*
# Should show: /usr/bin/node -> /usr/bin/node-22
```

### 2. Test Module Enable (RHEL8/9, C9S)

```bash
# Check Dockerfile has module enable command
grep "module enable nodejs" 22/Dockerfile.rhel8
grep "module enable nodejs" 22/Dockerfile.c9s
```

### 3. Test Build Dependencies

```bash
# RHEL8 should have gcc-toolset-13
grep "gcc-toolset-13" 22/Dockerfile.rhel8

# Fedora should have libatomic_ops
grep "libatomic_ops" 22/Dockerfile.fedora
```

## CI/CD Integration

The repository has GitHub Actions for automated testing:

```bash
# Check workflow files
ls .github/workflows/

# The build-and-push workflow should test all matrices
cat .github/workflows/build-and-push.yml
```

## Troubleshooting

If tests fail:

1. **Check submodules are initialized:**
   ```bash
   git submodule status
   ```

2. **Verify distgen installation:**
   ```bash
   distgen --version
   ```

3. **Compare generated vs committed files:**
   ```bash
   # Regenerate and diff
   make generate
   git diff
   ```

4. **Check for missing dependencies:**
   ```bash
   # For building images
   podman version
   # or
   docker version
   ```

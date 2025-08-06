# Greengage CI Build Image Docker Action

This GitHub Action builds and caches Docker images for Greengage CI/CD pipelines. It creates containerized environments based on specified Greengage versions and target operating systems, tags images with the commit SHA, and caches them for use in subsequent testing jobs. For pull requests within the same repository, it optionally pushes images to the GitHub Container Registry (GHCR) for debugging.

## Purpose

The `ci-build-img-docker` Action streamlines the creation of Docker images in Greengage CI pipelines. It supports flexible versioning and OS configurations, ensuring consistent environments for testing. The Action tags images with the commit SHA, caches them using GitHub Actions cache, and pushes developer-tagged images to GHCR for pull requests within the same repository, facilitating debugging.

## Usage

To use this Action in your workflow:

1. Add a job with a step that calls the Action from `greengagedb/ci-build-img-docker`.
2. Provide the required inputs.
3. Ensure the necessary permissions and Dockerfile are available.

### Inputs

| Name                | Description                                      | Required | Type   | Default |
|---------------------|--------------------------------------------------|----------|--------|---------|
| `version`           | Greengage version (e.g., `6` or `7`)             | Yes      | String | -       |
| `target_os`         | Target operating system (e.g., `ubuntu`, `centos`) | Yes    | String | -       |
| `target_os_version` | Target OS version (e.g., `22`, `7`)              | No       | String | `''`    |
| `python3`           | Python3 build argument for the Dockerfile        | No       | String | `''`    |

### Requirements

- **Permissions**:
  - `contents: read`: To checkout the repository.
  - `packages: write`: To push images to GHCR (for pull requests).
  - `actions: write`: To manage caching.
- **Environment Variables**: The Action uses the default `GITHUB_TOKEN` provided by GitHub Actions, which must have sufficient permissions for GHCR and cache operations.
- **Dockerfile**: Ensure a Dockerfile exists at `ci/Dockerfile.<target_os><target_os_version>` (e.g., `ci/Dockerfile.ubuntu`, `ci/Dockerfile.centos7`) in the calling repository.
- **Repository Access**: The Action clones the repository specified in `GITHUB_REPOSITORY`.

### Examples

#### Single Configuration

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      actions: write
    steps:
      - name: Build Docker Image
        uses: greengagedb/ci-build-img-docker@v1
        with:
          version: 7
          target_os: ubuntu
          target_os_version: ''
          python3: ''
```

#### Matrix Configuration

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: true
      matrix:
        target_os: [ubuntu, centos]
    permissions:
      contents: read
      packages: write
      actions: write
    steps:
      - name: Build Docker Image
        uses: greengagedb/ci-build-img-docker@v1
        with:
          version: 6
          target_os: ${{ matrix.target_os }}
          target_os_version: ''
          python3: ''
```

## Notes

- The Action clones the repository at the current commit SHA (or pull request head SHA) with full history and submodules.
- Images are tagged with the format `ghcr.io/<owner>/<repo>/ggdb<version>_<target_os><target_os_version>:<sha>` (e.g., `ghcr.io/greengagedb/greengage-ci/ggdb6_ubuntu:<sha>`).
- For pull requests within the same repository, a developer tag (sanitized branch name, e.g., `feature_branch`) is added and pushed to GHCR.
- The image is saved and cached using GitHub Actions cache with the key `ggdb<version>_<target_os><target_os_version>_<sha>`.
- Ensure the target OS and version match an existing Dockerfile in the `ci/` directory of the calling repository.
- For security, use tagged releases (e.g., `@v1`) instead of branch references.

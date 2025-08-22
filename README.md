# GitHub Release Manager

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-GitHub%20Release%20Manager-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAM6wAADOsB5dZE0gAAABl0RVh0U29mdHdhcmUAd3d3Lmlua3NjYXBlLm9yZ5vuPBoAAAERSURBVCiRhZG/SsMxFEZPfsVJ61jbxaF0cRQRcRJ9hlYn30IHN/+9iquDCOIsblIrOjqKgy5aKoJQj4n3EX8DyzAQ085GEISGc6/3cMMnk3uyEwO4dvVe4nT+VG9VaU98GTziOSvF+pfgSXqNuqVQKKdoT1+7UFELXLlPLOhINjLn5fXj3zkBXXsXPBvuXe8FpJ4GPKPQPdSKXJqQQB6zPDt2s4q8Z9q2Vz1UXKyOsQXkk3SJRGpuE0cPPGh+R7lPpJrHm/PX8VLp8/bk8Z6lDNSe/x6eSsNmqU7CdLEzfQLTJKLxuDyR1cX2SJWfAF5ddgCZYZlMV4dIa+lEF+CxjOOI6hrmQp2P/fqG4xyDZ2f/AAAAAElFTkSuQmCC)](https://github.com/marketplace/actions/github-release-manager)
[![CI](https://github.com/kubegrind/github-release-manager/workflows/CI/badge.svg)](https://github.com/kubegrind/github-release-manager/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A professional and feature-rich GitHub Action for creating and managing releases with comprehensive asset upload capabilities, automatic release notes generation, and extensive workflow integration.

##  Features

-  **Create or Update Releases**: Automatically create new releases or update existing ones
-  **Asset Management**: Upload multiple files using glob patterns with overwrite protection
-  **Release Notes**: Automatic generation from commits and pull requests
-  **Flexible Tagging**: Support for custom tags, auto-detection, and target commitish
-  **Workflow Integration**: Comprehensive outputs for downstream workflow steps
-  **Advanced Options**: Draft releases, prereleases, latest release management
-  **Rich Summaries**: Detailed workflow summaries with asset information
-  **Error Handling**: Robust error handling with detailed feedback

##  Quick Start

### Basic Usage

```yaml
name: Create Release
on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Create Release
        uses: kubegrind/github-release-manager@v1
        with:
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          body: |
            ## What's Changed
            - Bug fixes and improvements
            - New features added
          generate_release_notes: true
```

### Advanced Usage with Asset Upload

```yaml
name: Build and Release
on:
  push:
    tags:
      - 'v*'

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Artifacts
        run: |
          mkdir -p dist
          echo "Building application..." > dist/app.txt
          tar -czf dist/app-linux.tar.gz dist/app.txt
          zip -r dist/app-windows.zip dist/app.txt
      
      - name: Create Release with Assets
        uses: kubegrind/github-release-manager@v1
        with:
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          body_path: CHANGELOG.md
          files: |
            dist/*.tar.gz
            dist/*.zip
            LICENSE
            README.md
          generate_release_notes: true
          make_latest: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

##  Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `body` | Release description text | No | `''` |
| `body_path` | Path to file containing release description | No | - |
| `name` | Release name. Defaults to tag name if not specified | No | - |
| `tag_name` | Git tag name. Defaults to github.ref_name | No | - |
| `draft` | Create release as draft | No | `false` |
| `prerelease` | Mark release as prerelease | No | `false` |
| `files` | Newline-delimited list of file paths or glob patterns | No | - |
| `token` | GitHub token with contents write permission | No | `${{ github.token }}` |
| `repository` | Target repository in owner/repo format | No | Current repo |
| `target_commitish` | Commitish value for tag creation | No | Default branch |
| `generate_release_notes` | Auto-generate release notes from commits and PRs | No | `false` |
| `make_latest` | Set this release as the latest release | No | `true` |
| `overwrite_files` | Overwrite existing release assets with same name | No | `true` |
| `fail_on_unmatched_files` | Fail if file patterns don't match any files | No | `false` |

##  Outputs

| Output | Description |
|--------|-------------|
| `id` | Release ID |
| `url` | Release HTML page URL |
| `upload_url` | Release asset upload URL |
| `tag_name` | Git tag name used for the release |
| `assets` | JSON array of uploaded asset information |

## Usage Examples

### 1. Simple Release Creation

```yaml
- name: Create Simple Release
  uses: kubegrind/github-release-manager@v1
  with:
    tag_name: v1.0.0
    name: Version 1.0.0
    body: "Initial release"
```

### 2. Draft Release with Multiple Assets

```yaml
- name: Create Draft Release
  uses: kubegrind/github-release-manager@v1
  with:
    tag_name: v2.0.0-beta
    name: Beta Release v2.0.0
    draft: true
    prerelease: true
    files: |
      build/*.exe
      build/*.dmg
      build/*.deb
      docs/*.pdf
```

### 3. Release with Auto-Generated Notes

```yaml
- name: Release with Auto Notes
  uses: kubegrind/github-release-manager@v1
  with:
    tag_name: ${{ github.ref_name }}
    generate_release_notes: true
    files: |
      dist/app-*
      CHANGELOG.md
```

### 4. Cross-Repository Release

```yaml
- name: Create Release in Another Repo
  uses: kubegrind/github-release-manager@v1
  with:
    repository: myorg/another-repo
    tag_name: v1.0.0
    name: Cross-repo Release
    body: "Release created from another repository"
    token: ${{ secrets.CROSS_REPO_TOKEN }}
```

### 5. Using Release Outputs

```yaml
- name: Create Release
  id: create_release
  uses: kubegrind/github-release-manager@v1
  with:
    tag_name: v1.0.0
    files: dist/*

- name: Use Release Outputs
  run: |
    echo "Release ID: ${{ steps.create_release.outputs.id }}"
    echo "Release URL: ${{ steps.create_release.outputs.url }}"
    echo "Assets: ${{ steps.create_release.outputs.assets }}"
```

## 🔧 Advanced Configuration

### File Patterns

The `files` input supports various patterns:

```yaml
files: |
  # Specific files
  dist/app.exe
  README.md
  
  # Glob patterns
  dist/*.tar.gz
  build/**/*.zip
  
  # Multiple extensions
  artifacts/*.{exe,dmg,deb}
```

### Body from File

You can specify release notes from a file:

```yaml
- name: Release with Changelog
  uses: kubegrind/github-release-manager@v1
  with:
    body_path: RELEASE_NOTES.md
    # body input will be ignored if body_path is provided
```

### Conditional Release Creation

```yaml
- name: Conditional Release
  uses: kubegrind/github-release-manager@v1
  if: startsWith(github.ref, 'refs/tags/')
  with:
    tag_name: ${{ github.ref_name }}
    prerelease: ${{ contains(github.ref_name, 'beta') || contains(github.ref_name, 'alpha') }}
```

## Permissions

Make sure your workflow has the necessary permissions:

```yaml
permissions:
  contents: write  # Required for creating releases
  pull-requests: read  # Required for auto-generated release notes
```

## Error Handling

The action provides comprehensive error handling:

- **File Upload Failures**: Individual file failures won't stop the entire process
- **Pattern Matching**: Optional strict mode for unmatched file patterns
- **API Errors**: Detailed error messages for debugging
- **Workflow Summaries**: Rich summaries even when errors occur

## Workflow Integration

### With Build Matrix

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]

steps:
  - name: Build for ${{ matrix.os }}
    run: |
      # Build logic here
      
  - name: Create Release
    if: matrix.os == 'ubuntu-latest'
    uses: kubegrind/github-release-manager@v1
    with:
      files: dist/*
```

### With Artifact Download

```yaml
- name: Download Artifacts
  uses: actions/download-artifact@v4
  with:
    path: artifacts/

- name: Create Release
  uses: kubegrind/github-release-manager@v1
  with:
    files: artifacts/**/*
```

## Workflow Summary

The action automatically generates rich workflow summaries including:

- Release information and links
- Asset upload status and download links
- File sizes and download counts
- Error details when applicable

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- [Documentation](https://github.com/kubegrind/github-release-manager)
- [Report Issues](https://github.com/kubegrind/github-release-manager/issues)
- [Discussions](https://github.com/kubegrind/github-release-manager/discussions)

## Star History

If this action helped you, please consider giving it a star! ⭐

# GitHub Actions Workflows

This directory contains automated build and release workflows for AdaptixC2.

## Workflows

### 1. Build and Release (`build-release.yml`)

**Triggers:**
- When a tag matching `v*` is pushed (e.g., `v1.0.0`, `v2.1.3`)
- Manual workflow dispatch from GitHub Actions UI

**Jobs:**
1. **build-server** - Builds the AdaptixServer (teamserver) for Linux
   - Uses Go 1.25.4
   - Builds server and extenders
   - Packages as `adaptix-server-linux-amd64.tar.gz`

2. **build-client-linux** - Builds the AdaptixClient for Linux
   - Uses Qt6 and CMake
   - Packages as `adaptix-client-linux-amd64.tar.gz`

3. **build-client-macos** - Builds the AdaptixClient for macOS
   - Uses Qt6 and CMake via Homebrew
   - Packages as `adaptix-client-macos-arm64.tar.gz`
   - Built on macOS ARM64 (Apple Silicon)

4. **build-client-windows** - Builds the AdaptixClient for Windows
   - Uses Qt6 with MinGW
   - Includes Qt dependency deployment via windeployqt
   - Packages as `adaptix-client-windows-amd64.zip`
   - **Resilient to errors**: Continues and uploads artifacts even if build fails

5. **release** - Creates GitHub Release
   - Only runs when a tag is pushed
   - Uploads all build artifacts to the release
   - Generates release notes automatically

### 2. CI Build (`ci.yml`)

**Triggers:**
- Push to `main` or `dev` branches
- Pull requests to `main` or `dev` branches

**Jobs:**
1. **build-server** - Validates server build
2. **build-client-linux** - Validates Linux client build

## Usage

### Creating a Release

To create a new release with all platform builds:

1. Create and push a tag:
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

2. GitHub Actions will automatically:
   - Build server for Linux
   - Build clients for Linux, macOS, and Windows
   - Create a GitHub Release with all artifacts
   - Generate release notes

3. The release will be available at: `https://github.com/keac/AdaptixC2/releases`

### Manual Workflow Trigger

You can also manually trigger the build and release workflow:

1. Go to the "Actions" tab in GitHub
2. Select "Build and Release" workflow
3. Click "Run workflow"
4. Choose the branch and click "Run workflow"

Note: Manual runs won't create a release unless running from a tag.

## Build Artifacts

Each successful build produces the following artifacts:

| Platform | Artifact Name | Contents |
|----------|--------------|----------|
| Server (Linux) | `adaptix-server-linux-amd64.tar.gz` | Server binary, extenders, SSL scripts, config files |
| Linux Client | `adaptix-client-linux-amd64.tar.gz` | AdaptixClient executable |
| macOS Client | `adaptix-client-macos-arm64.tar.gz` | AdaptixClient executable |
| Windows Client | `adaptix-client-windows-amd64.zip` | AdaptixClient.exe + Qt DLLs |

## Security

All workflows follow GitHub Actions security best practices:
- Minimal permissions granted to GITHUB_TOKEN
- Build jobs have read-only access
- Release job has write access only for creating releases
- Dependencies pinned to specific versions where possible

## Dependencies

### Server Build
- Go 1.25.4
- gcc, g++, make
- mingw-w64

### Linux Client Build
- gcc, g++, make, cmake
- Qt6 (base, websockets, declarative)
- OpenSSL

### macOS Client Build
- cmake, make
- Qt6 (via Homebrew)
- OpenSSL (via Homebrew)

### Windows Client Build
- Qt6 6.7.3
- MinGW (included with Qt installer)
- CMake (included with Qt installer)

## Troubleshooting

If builds fail:

1. Check the Actions tab for detailed logs
2. Verify dependencies are correctly specified in workflow files
3. Ensure Makefile targets work correctly
4. For Windows builds, verify MinGW and Qt paths are correct

**Note on Windows builds**: The Windows build job is configured to continue and upload artifacts even if errors occur during the build process. This ensures that:
- Other platform builds (Linux, macOS) can complete successfully
- Partial build artifacts are uploaded when possible
- The release process can proceed with available platform builds

If a Windows build fails, check the uploaded artifact - it may contain a `BUILD_FAILED.txt` file indicating no executable was built.

For local testing:
- Use `make server` to build the server
- Use `make client` to build the client
- Refer to Makefile for other build targets

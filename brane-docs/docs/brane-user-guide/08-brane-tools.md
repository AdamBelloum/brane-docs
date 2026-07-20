# 8. Brane Toolchain Installation Guide

This section covers the setup and installation of the two primary command-line utilities used within the Brane ecosystem:

*   **`brane` (Brane CLI):** The end-user interface used by scientists, data engineers, and software developers to build packages, manage datasets, and test workflows.
*   **`branectl` (Brane Control):** The administrative toolkit used by system engineers and cluster operators to provision nodes, manage cluster secrets, and generate access tokens.

---

## 8.1. Runtime Dependencies

Before downloading either tool, ensure your target host machine meets the following baseline dependencies:

| Component | Description | Verification / Installation |
|---|---|---|
| **GLIBC $\ge$ 2.27** | Core system library required by precompiled binaries | Run `ldd --version` to verify compatibility. |
| **OpenSSL** | Required for cryptographic handshakes and token validation | Ubuntu/Debian: `sudo apt install openssl`<br>macOS: `brew install openssl` |

---

## 8.2. Installing the Brane CLI (`brane`)

The `brane` executable is required by any user who plans to compile packages or manage local data definitions.

<Sequence>
{/* Reason: Procedural installation steps where misordering (e.g., executing before permissions are set) causes system failures. */}
  <Step title="Download the Binary" subtitle="Step 1">
    Fetch the precompiled binary from the official releases page. Replace `vX.Y.Z` with your targeted production version number:
    ```bash
    # Download the binary
    curl -L -o brane [https://github.com/brane-framework/brane/releases/download/vX.Y.Z/brane](https://github.com/brane-framework/brane/releases/download/vX.Y.Z/brane)

    # Make the binary executable
    chmod +x brane
    ```
  </Step>
  <Step title="Install to System Path" subtitle="Step 2">
    Move the executable to a global directory so it can be invoked from any terminal workspace:
    ```bash
    sudo mv brane /usr/local/bin/
    ```
  </Step>
  <Step title="Verify the Installation" subtitle="Step 3">
    Confirm that the binary is globally accessible and running correctly by checking its version:
    ```bash
    brane --version
    ```
  </Step>
</Sequence>


---
---

## 8.3. Installing the Administrative CLI (`branectl`)

`branectl` must be installed on the host machines where Brane infrastructure services (central or worker nodes) are being deployed or managed.

### Step 1: Download and Route the Target Binary
Select the command block matching your host operating system architecture. Replace `<BRANE_VERSION>` with your targeted release tag (e.g., `v3.0.0` or `nightly`):

*   **Linux (x86-64):**
    ```bash
    sudo wget -O /usr/local/bin/branectl [https://github.com/BraneFramework/brane/releases/download/](https://github.com/BraneFramework/brane/releases/download/)<BRANE_VERSION>/branectl-linux-x86_64
    ```
*   **macOS (Apple Silicon / aarch64):**
    ```bash
    sudo wget -O /usr/local/bin/branectl [https://github.com/BraneFramework/brane/releases/download/](https://github.com/BraneFramework/brane/releases/download/)<BRANE_VERSION>/branectl-macos-aarch64
    ```

### Step 2: Apply Permissions and Verify
Make the administrative binary executable and check its integrated helper utility to confirm a successful setup:

```bash
# Grant executable permissions
sudo chmod +x /usr/local/bin/branectl

# Verify the CLI tool environment
branectl --help

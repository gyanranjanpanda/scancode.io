# ScanCode.io SCA Integration Fixtures

This directory contains SBOM fixtures used to test `ScanCode.io`'s integrations with various Software Composition Analysis (SCA) tools. These fixtures ensure the `load_sbom` pipeline accurately ingests and maps the results from these tools into the ScanCode.io data model.

## Included Fixtures

These fixtures are **authentic, tool-generated SBOMs**. They were created using the actual corresponding tools to guarantee realistic schema variations and quirks.

### 1. CycloneDX Python
* **Tool Name:** `cyclonedx-py` (part of `cyclonedx-bom`)
* **Tool Version:** 7.0.1-alpha.2 (`cyclonedx-python-lib` 10.4.1)
* **Format:** CycloneDX 1.6 JSON
* **Generation Command:**
  ```bash
  # Inside a sample python virtual environment
  pip install cyclonedx-bom
  cyclonedx-py environment --output-format JSON --outfile cyclonedx-python-sbom.json
  ```

### 2. CycloneDX Node.js npm
* **Tool Name:** `@cyclonedx/cyclonedx-npm`
* **Tool Version:** 4.1.2 (`cyclonedx-library` 9.4.1)
* **Format:** CycloneDX 1.6 JSON
* **Generation Command:**
  ```bash
  mkdir sample-app && cd sample-app
  npm init -y && npm install lodash@4.17.21 --package-lock-only
  npx @cyclonedx/cyclonedx-npm --package-lock-only --output-format JSON --output-file cyclonedx-node-npm-sbom.json
  ```
* **Note:** The fixture was intentionally kept small (1 package) to maintain a minimal and deterministic test footprint.

### 3. apko
* **Tool Name:** `apko` (Chainguard OCI image builder)
* **Tool Version:** 1.1.11
* **Format:** SPDX 2.3 JSON
* **Generation Command:**
  ```bash
  # Using an apko.yaml containing alpine-baselayout, busybox, ca-certificates-bundle, musl, zlib
  docker run --rm -v $(pwd):/workspace -v $(pwd)/sbom-out:/sbom-out \
      cgr.dev/chainguard/apko:latest build --sbom-path /sbom-out \
      /workspace/apko-test.yaml apko-test-image:latest /dev/null
  ```

---

## Fixture Selection Rationale & Excluded Tools

`ScanCode.io` generically supports CycloneDX and SPDX standard formats. Several tools generate compatible SBOMs, but not all of them require explicit fixtures in this test suite.

The following tools are listed as **supported** in the documentation because their output conforms to one of the above standards, but **fixtures are deliberately excluded** from this test suite:

1. **Aikido Security Scanner**
2. **Amazon Inspector SBOM Generator (Sbomgen)**
3. **gh-sbom (GitHub CLI SBOM extension)**

**Reasoning:**
Fixtures are included only for tools where authentic SBOMs can be generated deterministically and locally using open, publicly available tooling (e.g., Docker images or `pip`/`npm` packages).
Generating authentic SBOMs for Aikido, Sbomgen, or `gh-sbom` often requires proprietary environments, cloud accounts, or authenticated dashboards. Including synthetic (fake) fixtures or using alternative tools (like Trivy or Grype) and relabeling them was evaluated and rejected to ensure the test data remains strictly authentic and trustworthy. Since these excluded tools simply output standard CycloneDX or SPDX, the existing generic pipeline tests (plus the authentic fixtures from Anchore, Trivy, OSV-Scanner, etc.) already validate their structure.

Additionally:
* **SCANOSS:** Excluded because its output relies on a proprietary JSON schema (WDPE) instead of a standard generic SBOM format (CycloneDX/SPDX). Ingesting SCANOSS logic requires building a dedicated extraction pipeline rather than using the generic `load_sbom` pipeline. This is tracked in a separate issue.

# debezium-descriptors-registry
Official registry of JSON configuration schemas for all Debezium connectors (MySQL, PostgreSQL, MongoDB, Oracle, SQL Server, etc.), transformations (SMTs), and sink connectors. Organized by release version with automated snapshot builds. Powers documentation, tooling, validation, and OCI artifacts.

## Repository Structure

The repository is organized by Debezium release versions, with each version directory containing configuration descriptors for all components:

```
debezium-descriptors-registry/
├── 3.4.0-Final/
│   ├── manifest.json                    # Version metadata and component index
│   ├── source-connector/                # Source connector descriptors
│   │   ├── io.debezium.connector.mysql.MySqlConnector.json
│   │   ├── io.debezium.connector.postgresql.PostgresConnector.json
│   │   └── ...
│   ├── sink-connector/                  # Sink connector descriptors
│   │   ├── io.debezium.connector.jdbc.JdbcSinkConnector.json
│   │   └── ...
│   └── transformation/                  # SMT (Single Message Transform) descriptors
│       ├── io.debezium.transforms.ExtractNewRecordState.json
│       └── ...
├── 3.4.1-Alpha1/
│   └── ... (same structure)
├── 3.4.1-Beta1/
│   └── ... (same structure)
└── .github/
    ├── workflows/                       # GitHub Actions workflows
    └── actions/                         # Reusable custom actions
```

### Manifest File

Each version directory contains a `manifest.json` file with:
- **Schema version**: Format version of the manifest
- **Build metadata**: Timestamp, source repository, source commit, source branch
- **Component index**: Lists all connectors and transformations with their descriptors

Example structure:
```json
{
  "schemaVersion": "1.0",
  "build": {
    "timestamp": "2026-01-05T13:08:59Z",
    "sourceRepository": "debezium/debezium",
    "sourceCommit": "758efa135",
    "sourceBranch": "dbz#1544"
  },
  "components": {
    "source-connector": [...],
    "sink-connector": [...],
    "transformation": [...]
  }
}
```

## Branching Strategy

### `main` Branch (Release)
- Contains stable release versions (e.g., `3.4.0-Final`, `3.4.1-Beta1`)
- New version directories are added when Debezium releases are published
- Each push triggers OCI artifact publication with version-specific tags

### `snapshot` Branch (Nightly)
- Contains development/snapshot versions (e.g., `3.5.0-SNAPSHOT`)
- Automatically updated with latest development builds
- Publishes nightly OCI artifacts for testing and early access

## Automated Build System

### GitHub Actions Workflows

#### 1. Release Build (`build-oci-artifact-main.yml`)
**Trigger**: Push to `main` branch

**Process**:
1. Detects new or changed version directories by analyzing git diff
2. Extracts metadata from the version's `manifest.json`
3. Creates OCI artifact bundle (excludes `.git` and `.github`)
4. Pushes artifact to `quay.io/debezium/debezium-descriptors` with:
   - Version-specific tag (e.g., `:3.4.0-Final`)
   - `:latest` tag (always points to most recent release)

#### 2. Nightly Build (`build-oci-artifact.yml`)
**Trigger**: Push to `snapshot` branch

**Process**:
1. Finds the snapshot version's `manifest.json`
2. Extracts metadata from manifest
3. Creates OCI artifact bundle
4. Pushes artifact to `quay.io/debezium/debezium-descriptors:nightly`

### Custom GitHub Actions

The workflows use reusable composite actions located in `.github/actions/`:

- **`extract-manifest-metadata`**: Parses `manifest.json` to extract version, source repository, commit hash, and timestamps
- **`setup-oras`**: Installs and configures [ORAS CLI](https://oras.land/) for OCI artifact operations
- **`login-registry`**: Authenticates to container registry (Quay.io)
- **`push-oci-artifact`**: Pushes OCI artifacts with standardized metadata annotations

## OCI Artifact Distribution

Configuration descriptors are distributed as OCI artifacts for easy consumption by tooling and automation.

### Registry Information

- **Registry**: `quay.io`
- **Repository**: `debezium/debezium-descriptors`
- **Artifact Type**: `application/vnd.debezium.descriptors.v1+json`

### Pulling OCI Artifacts

Using ORAS CLI:
```bash
# Pull latest release
oras pull quay.io/debezium/debezium-descriptors:latest

# Pull specific version
oras pull quay.io/debezium/debezium-descriptors:3.5.0-Final

# Pull nightly snapshot
oras pull quay.io/debezium/debezium-descriptors:nightly
```

### OCI Artifact Metadata

Each artifact includes comprehensive annotations:
- **Standard OCI annotations**: `org.opencontainers.image.*` (created, version, revision, source, documentation)
- **Debezium-specific annotations**: `io.debezium.*` (source repository, source commit, build timestamp)

Inspect artifact metadata:
```bash
oras manifest fetch quay.io/debezium/debezium-descriptors:latest
```

## Usage

### For Tooling and Automation
1. Pull the OCI artifact for your target version
2. Parse `manifest.json` to discover available components
3. Load individual descriptor files for validation or code generation

### For Documentation
- Version directories serve as the source of truth for configuration options
- Descriptors power auto-generated documentation on debezium.io

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

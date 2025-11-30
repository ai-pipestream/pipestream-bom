# Pipeline BOM (Bill of Materials)

[![Publish Snapshot](https://github.com/ai-pipestream/pipestream-bom/actions/workflows/snapshot.yml/badge.svg)](https://github.com/ai-pipestream/pipestream-bom/actions/workflows/snapshot.yml)
[![Publish Release](https://github.com/ai-pipestream/pipestream-bom/actions/workflows/release.yml/badge.svg)](https://github.com/ai-pipestream/pipestream-bom/actions/workflows/release.yml)

## Overview

This Bill of Materials (BOM) manages dependency versions across all Pipeline Engine microservices. It provides a centralized version catalog and ensures consistency across the entire Pipestream AI platform.

## Artifacts

This project publishes two artifacts:

| Artifact | Description |
|----------|-------------|
| `ai.pipestream:pipeline-bom` | Bill of Materials for dependency management |
| `ai.pipestream:pipeline-bom-catalog` | Gradle Version Catalog for use in `settings.gradle` |

## Version Strategy

### Quarkus First
We align with the latest stable Quarkus release and favor Quarkus-managed dependency versions wherever possible. This ensures compatibility with Quarkus's dependency model and tested integration.

### gRPC & Protocol Buffers
We use the **highest supported stable versions** of:
- **gRPC** - Latest stable release for optimal performance and features
- **Protocol Buffers (protobuf)** - Matched to gRPC compatibility requirements
- **Connect-ES** - Latest for TypeScript/JavaScript clients

These versions override Quarkus BOM versions using `strictly()` to ensure consistent gRPC stack across all services.

### Apicurio Registry
We track the **latest stable Apicurio versions** for:
- Schema Registry integration
- Protobuf schema serialization/deserialization
- Registry client SDKs

### Other Key Dependencies
- **Kafka** - Aligned with Quarkus recommendations
- **OpenSearch** - Latest stable Java client
- **WireMock** - Latest for integration testing with gRPC support
- **Security** - password4j for password hashing
- **Testing** - SmallRye Reactive Messaging In-Memory for testing

## Publishing Locations

This BOM is published to multiple locations:

| Location | URL | Type |
|----------|-----|------|
| Maven Central | https://central.sonatype.com | Releases |
| Maven Central Snapshots | https://central.sonatype.com/repository/maven-snapshots/ | Snapshots |
| GitHub Packages | https://maven.pkg.github.com/ai-pipestream/pipestream-bom | All versions |
| Local Maven | `~/.m2/repository` | Local development |

## Usage

### Repository Configuration

Add the snapshot repository to your project if you need SNAPSHOT versions:

```gradle
// settings.gradle
dependencyResolutionManagement {
    repositories {
        mavenCentral()
        
        // For SNAPSHOT versions
        maven {
            url = uri('https://central.sonatype.com/repository/maven-snapshots/')
            mavenContent {
                snapshotsOnly()
            }
            content {
                includeGroupByRegex "ai\\.pipestream(\\..*)?"
            }
        }
    }
}
```

### Using the BOM

```gradle
// build.gradle
dependencies {
    implementation platform('ai.pipestream:pipeline-bom:0.4.0-SNAPSHOT')
    
    // Now you can omit versions - they're managed by the BOM
    implementation 'io.quarkus:quarkus-grpc'
    implementation 'io.grpc:grpc-protobuf'
    implementation 'io.grpc:grpc-stub'
}
```

### Using the Version Catalog

Import the catalog in your `settings.gradle`:

```gradle
// settings.gradle
dependencyResolutionManagement {
    versionCatalogs {
        libs {
            from("ai.pipestream:pipeline-bom-catalog:0.4.0-SNAPSHOT")
        }
    }
}
```

Then use it in your `build.gradle`:

```gradle
// build.gradle
dependencies {
    implementation libs.quarkus.grpc
    implementation libs.grpc.protobuf
    implementation libs.grpc.stub
}
```

### Pipestream Internal Dependencies

The BOM manages versions for all Pipestream internal libraries:

```gradle
dependencies {
    implementation platform('ai.pipestream:pipeline-bom:0.4.0-SNAPSHOT')
    
    // Internal libraries - versions managed by BOM
    implementation 'ai.pipestream:pipeline-commons'
    implementation 'ai.pipestream:pipeline-api'
    implementation 'ai.pipestream:dynamic-grpc'
    implementation 'ai.pipestream:grpc-stubs'
}
```

## Maintenance

### Updating Versions

When updating versions:
1. Check Quarkus release notes for dependency updates
2. Verify gRPC/protobuf compatibility matrix
3. Test with integration test suite before publishing
4. Update this README if version strategy changes

### Dependency Updates

This repository uses [Dependabot](https://docs.github.com/en/code-security/dependabot) to automatically:
- Create PRs for dependency updates
- Group related dependencies (Quarkus, gRPC, testing, etc.)
- Auto-merge patch updates for stable dependencies

## CI/CD

### Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| Snapshot | Push to `main`, nightly at 3 AM UTC | Publish SNAPSHOT to Maven Central & GitHub Packages |
| Release | Tag `v*` or manual dispatch | Publish release to Maven Central & GitHub Packages |

### Required Secrets

The following secrets must be configured in the repository settings:

| Secret | Description |
|--------|-------------|
| `MAVEN_CENTRAL_USERNAME` | Maven Central (Sonatype) username |
| `MAVEN_CENTRAL_PASSWORD` | Maven Central (Sonatype) password |
| `GPG_PRIVATE_KEY` | GPG private key for signing artifacts |
| `GPG_PASSPHRASE` | Passphrase for the GPG private key |

Note: `GITHUB_TOKEN` is automatically provided by GitHub Actions.

### Creating a Release

1. **Automatic (recommended)**: Go to Actions → "Publish Release" → Run workflow → Select version bump type
2. **Manual**: Create and push a tag matching `v*` pattern (e.g., `v0.4.0`)

## Building Locally

```bash
# Build the BOM
./gradlew build

# Publish to local Maven repository
./gradlew publishToMavenLocal

# Check current version
./gradlew currentVersion

# Create a release tag (if on main)
./gradlew createRelease
```

## Project Structure

```
pipestream-bom/
├── build.gradle              # Main build configuration
├── settings.gradle           # Project settings
├── gradle/
│   ├── libs.versions.toml    # Version catalog source
│   └── wrapper/              # Gradle wrapper files
├── .github/
│   ├── dependabot.yml        # Dependabot configuration
│   └── workflows/
│       ├── release.yml       # Release workflow
│       └── snapshot.yml      # Snapshot workflow
├── LICENSE                   # MIT License
└── README.md                 # This file
```

## License

MIT License - see [LICENSE](LICENSE) for details.

## Related Projects

- [platform-libraries](https://github.com/ai-pipestream/platform-libraries) - Core platform libraries
- [pipestream-protos](https://github.com/ai-pipestream/pipestream-protos) - Protocol Buffer definitions

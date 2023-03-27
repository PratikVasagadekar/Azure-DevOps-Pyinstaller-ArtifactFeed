# Untitled

## Azure Pipeline to Build Pyinstaller Executable (EXE), Publish it as Pipeline Artifact, and Push it to Azure Artifacts Feed.

---

### Trigger :

```yaml
trigger:
  branches:
    include:
      - release
  tags:
    include:
      - release*
  paths:
    exclude:
      - /*
```

Trigger is based on Certain Tag and Will only work for Branch named release

### Host Agent :

```yaml
pool:
  name: Default
```

Used Self Hosted Windows 2019 agent for Build Execution.

### Building EXE with Pyinstaller

```yaml
strategy:
  matrix:
    Python310:
      python.version: '3.10'

steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.11'
    architecture: 'x64'
- script: pip install -r requirements.txt
  displayName: 'Install dependencies'

- script: |
    pyinstaller -y demo.spec
  displayName: 'Build pyInstaller Application'
```

PyInstaller will need a **Spec** file for building an EXE out of Source.

### Archiving, Publishing and Pushing to Feeds

```yaml
- task: ArchiveFiles@2
  inputs:
    rootFolderOrFile: '$(Build.SourcesDirectory)\dist'
    includeRootFolder: false 
    archiveType: 'zip'
    archiveFile: '$(Build.ArtifactStagingDirectory)\dist\release.zip'
  displayName: 'Archive and Copy to Artifact Staging'

- task: UniversalPackages@0
  inputs:
    command: 'publish'
    publishDirectory: '$(Build.ArtifactStagingDirectory)\dist'
    feedsToUse: 'internal'
    vstsFeedPublish: 'releases'
    packagePublishDescription: 'My published artifacts'
    vstsPackageVersion: 'v1.0.0'
    vstsFeedPackagePublish: 'release'

- task: PublishBuildArtifacts@1
  inputs:
    artifactName: '$(Agent.OS)'
    pathtoPublish: '$(Build.ArtifactStagingDirectory)\dist\release.zip'
```

1. ArchiveFiles will zip the Dist folder.
2. UniversalPackages will Publish the Build Artifact to Azure Artifacts Feed.
3. PublishBuildArtifacts will publish the Build to Pipeline Artifact****

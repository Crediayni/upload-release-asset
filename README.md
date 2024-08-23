# upload-release-asset

This action can be used to upload an asset to the release.  This action is based on GitHub's [upload-release-asset] action which has been deprecated.  The [Crediayni/create-release] action can also upload a file as a release asset but this action may be used when the asset is not available when the release is created or if multiple files need to be uploaded as assets in the release.

This action needs an upload url which an [api call] will return in the response or the [Crediayni/create-release] action will return as an output.

## Index

- [upload-release-asset](#upload-release-asset)
  - [Index](#index)
  - [Inputs](#inputs)
  - [Outputs](#outputs)
  - [Usage Examples](#usage-examples)
  - [Contributing](#contributing)
    - [Recompiling](#recompiling)
    - [Incrementing the Version](#incrementing-the-version)
   
## Inputs
| Parameter            | Is Required | Description                                                                                                                                        |
| -------------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `github-token`       | true        | A token with permission to create and delete releases.  Generally secrets.GITHUB_TOKEN.                                                            |
| `upload-url`         | true        | The URL for uploading assets to the release.  Can be taken from the outputs of an api call to create a release like in [Crediayni/create-release]. |
| `asset-path`         | true        | The path to the asset you want to upload.                                                                                                          |
| `asset-name`         | true        | The name of the asset you want to upload.                                                                                                          |
| `asset-content-type` | true        | The content-type of the asset you want to upload. See the [supported Media Types].                                                                 |

## Outputs
| Output                       | Description                                                       |
| ---------------------------- | ----------------------------------------------------------------- |
| `asset-browser-download-url` | URL users can navigate to in order to download the uploaded asset |

## Usage Examples

```yml
on: 
  pull_request:
    types: [opened, reopened, synchronize]

jobs:
  create-release:
    runs-on: ubuntu-latest
    outputs:
      upload_url: ${{ steps.create_release.outputs.asset-upload-url }}

    steps:
      - uses: actions/checkout@v3
        with: 
          fetch-depth: 0

      - name: Calculate next version
        id: version
        uses: Crediayni/git-version-lite@v2.1.2
        with:
          calculate-prerelease-version: true
          branch-name: ${{ github.head_ref }}

      - name: Create release
        id: create_release
        uses: Crediayni/create-release@v3.1.1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          tag-name: ${{ steps.version.outputs.VERSION }}

  build-and-test:
    runs-on: ubuntu-latest
    needs: [create-release]
    env:
      PROJECT_ROOT: './src/MyProj'
      DEPLOY_ZIP: 'published_app.zip'
    steps:
      - name: Build, Publish and Zip App
        working-directory: ${{ env.PROJECT_ROOT }}
        run: |
          dotnet publish -c Release -o ./published_app 
          (cd published_app && zip -r ../${{env.DEPLOY_ZIP}} .)
      - name: Test
        ...
      
      - name: Upload published artifact
        uses: Crediayni/upload-release-asset@v1.1.1
        with: 
          github-token: ${{ secrets.GITHUB_TOKEN }}
          upload-url: ${{ needs.create-release.outputs.upload_url }}
          asset-path: ${{ env.PROJECT_ROOT }}/${{ env.DEPLOY_ZIP }}
          asset-name: ${{ env.DEPLOY_ZIP }}
          asset-content-type: application/zip
          
      

```

## Contributing

When creating new PRs please ensure:
1. The action has been recompiled.  See the [Recompiling](#recompiling) section below for more details.
2. For major or minor changes, at least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version](#incrementing-the-version).
3. The `README.md` example has been updated with the new version.  See [Incrementing the Version](#incrementing-the-version).
4. The action code does not contain sensitive information.

### Recompiling

If changes are made to the action's code in this repository, or its dependencies, you will need to re-compile the action.

```sh
# Installs dependencies and bundles the code
npm run build

# Bundle the code (if dependencies are already installed)
npm run bundle
```

These commands utilize [esbuild](https://esbuild.github.io/getting-started/#bundling-for-node) to bundle the action and
its dependencies into a single file located in the `dist` folder.

### Incrementing the Version

This action uses [git-version-lite] to examine commit messages to determine whether to perform a major, minor or patch increment on merge.  The following table provides the fragment that should be included in a commit message to active different increment strategies.
| Increment Type | Commit Message Fragment                     |
| -------------- | ------------------------------------------- |
| major          | +semver:breaking                            |
| major          | +semver:major                               |
| minor          | +semver:feature                             |
| minor          | +semver:minor                               |
| patch          | *default increment type, no comment needed* |

[git-version-lite]: https://github.com/Crediayni/git-version-lite
[upload-release-asset]: https://github.com/actions/upload-release-asset
[Crediayni/create-release]: https://github.com/Crediayni/create-release
[api call]: https://docs.github.com/en/rest/reference/repos#create-a-release
[supported Media Types]: https://www.iana.org/assignments/media-types/media-types.xhtml
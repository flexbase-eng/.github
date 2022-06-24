# Flexbase github resources

* [Workflows](#workflows)
  * [typescript.build](#typescript.build)
  * [typescript.sonarcloud](#typescript.sonarcloud)
  * [typescript.publish.npm](#typescript.publish.npm)

---

## <a id="workflows"></a> Workflows

### <a id="typescript.build"></a> typescript.build

Builds typescript using [yarn](https://yarnpkg.com/)

#### Usage
```
jobs:
  build:
    uses: flexbase-eng/.github/.github/workflows/typescript.build.yml@main
    with:
      package_folder: output
      revision: ${{github.run_number}}
```

#### Inputs

Input | Type | Default | Required
--- | --- | --- | ---
package_folder | `string` |  | no
use_packr | `boolean` | false | no
major_version[^1] | `integer` |  | no
minor_version[^1]  | `integer` |  | no
revision[^1]  | `integer` |  | no

[^1]: Packr must be used for version support

#### Steps

1. Yarn install
2. Yarn build
3. Yarn lint
4. Yarn test
5. Packr (`use_packr` but be set to `true`)
6. Store build artifacts
7. Store coverage artifacts

---

### <a id="typescript.sonarcloud"></a> typescript.sonarcloud

Pushes coverage to [sonarcloud](https://sonarcloud.io/)

#### Usage
```
  coverage:
    needs: build
    uses: flexbase-eng/.github/.github/workflows/typescript.sonarcloud.yml@main
    with:
      project_key: flexbase-eng_<repo-name>
    secrets: inherit
```

#### Inputs

Input | Type | Default | Required
--- | --- | --- | ---
project_key | `string` |  | *

#### Steps

1. Retreive coverage artifacts
2. Run sonarcloud

---

### <a id="typescript.publish.npm"></a> typescript.publish.npm

Pushes node package to [npm](https://www.npmjs.com/) under the `@flexbase` scope and organization

#### Usage
```
  publish:
    needs: coverage
    uses: flexbase-eng/.github/.github/workflows/typescript.publish.yml@main
    with:
      tag: beta
    secrets: inherit
```

#### Inputs

Input | Type | Default | Required
--- | --- | --- | ---
tag | `string` | latest | 

#### Steps

1. Retreive build artifacts
2. Setup yarn
3. Publish to npm

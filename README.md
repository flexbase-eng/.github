# Flexbase github resources

* [Workflows](#workflows)
  * [typescript.build](#typescript.build)
  * [typescript.sonarcloud](#typescript.sonarcloud)
  * [typescript.publish.npm](#typescript.publish.npm)
  * [typescript.release.notifier](#typescript.release.notifier)

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

### <a id="typescript.release.notifier"></a> typescript.release.notifier
Workflow to utilize a common pattern for release notifications when publishing a release via github.  Notifier aggregates the release notes and pushes this to a release channel. `gha_deployment_failure_url` defines the URL you would like to set for log review of the production deployment failure.

#### Usage
```
  prod-notify:
    needs: <production deploy worfklow name>
    uses: flexbase-eng/.github/.github/workflows/typescript.release.notifier.yml@main
    with:
      gha_deployment_failure_url: https://github.com/flexbase-eng/flexbase-web/actions/workflows/deploy-prod.yml
    secrets: inherit
```

#### Inputs

Input | Type | Default | Required
--- | --- | --- | ---
gha_deployment_failure_url | `string` | none | yes

#### Steps

Assumptions: production deployment triggered based on published release notes.  Deployment has completed (either succeeded or failed). 

1. Get the repositories latest published release
2. if success, post slack notification to release channel
3. if failure, post slack notificaiton to release channel with the `gha_deployment_failure_url` to help developers identify root cause.

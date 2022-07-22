# Flexbase github resources

* [Workflows](#workflows)
  * [typescript.build](#typescript.build)
  * [typescript.sonarcloud](#typescript.sonarcloud)
  * [typescript.publish.npm](#typescript.publish.npm)
  * [release.notifier](#release.notifier)
  * [release.drafter](#release.drafter)
  * [run.postman.api.tests](#run.postman.api.tests)

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

Pushes node package to [npm](https://www.npmjs.com/) under the `@flexbase` scope and organization.
Takes the `package.json` value and utilizes this with the standard npm publish methodology.
Optional `beta` tag as seen below for PR based publishes, to publish a beta version before main branch publish. This is useful if you'd like to test your changes within the package before making a formal release with merge to `main`.

#### Usage
publishing a beta on PR with the `npm` package management system: 
```
  publish:
    needs: coverage
    uses: flexbase-eng/.github/.github/workflows/typescript.publish.yml@main
    with:
      tag: beta
      package_manager: npm
    secrets: inherit
```

publishing to production with the `yarn` package management system: 
```
  publish:
    needs: coverage
    uses: flexbase-eng/.github/.github/workflows/typescript.publish.yml@main
    secrets: inherit
```

#### Inputs

Input | Type | Default | Required
--- | --- | --- | ---
tag | `string` | latest | no
build_artifact | `string` | latest | no
package_manager | `string` | yarn | no
package_visibility | `string` | public | no

#### Steps

1. Retreive build artifacts
2. Setup yarn
3. Publish to npm

---

### <a id="release.notifier"></a> release.notifier
Workflow to utilize a common pattern for release notifications when publishing a release via github.  Notifier aggregates the release notes and pushes this to a release channel. `gha_deployment_failure_url` defines the URL you would like to set for log review of the production deployment failure.

#### Usage
Entire workflow for reference to add to indidivual repository.
```
  name: Notify Slack Release Channel
    on:
    release:
      types: [released]
    workflow_dispatch:
  prod-notify:
    needs: <production deploy worfklow name>
    uses: flexbase-eng/.github/.github/workflows/release.notifier.yml@main
    with:
      gha_deployment_failure_url: https://github.com/flexbase-eng/flexbase-web/actions/workflows/deploy-prod.yml
      production_url: https://www.flexbase.app
      release_title: Flexbase Web
    secrets: inherit
```
or minimally: 
```
  name: Notify Slack Release Channel
    on:
    release:
      types: [released]
    workflow_dispatch:
  prod-notify:
    needs: <production deploy worfklow name>
    uses: flexbase-eng/.github/.github/workflows/release.notifier.yml@main
    with:
      gha_deployment_failure_url: https://github.com/flexbase-eng/flexbase-web/actions/workflows/deploy-prod.yml
    secrets: inherit
```

#### Inputs

Input | Type | Default | Required
--- | --- | --- | ---
gha_deployment_failure_url | `string` | none | yes
production_url | `string` | https://www.flexbase.app | no
release_title | `string` | repository.name | no

#### Steps

Assumptions: production deployment triggered based on published release notes.  Deployment has completed (either succeeded or failed). 

1. Get the repositories latest published release
2. if success, post slack notification to release channel
3. if failure, post slack notificaiton to release channel with the `gha_deployment_failure_url` to help developers identify root cause.

---

### <a id="release.drafter"></a> release.drafter
Drafts merged PR's into a github release draft format, aggregating labels to various sections.

#### Usage
Entire worfklow for reference to add to individual repository.
```
  name: Release draft workflow
  on:
    push:
      branches:
        - main
    # pull_request event is required only for autolabeler
    pull_request:
      # Only following types are handled by the action, but one can default to all as well
      types: [opened, reopened, synchronize]
  update_release_draft:
    uses: flexbase-eng/.github/.github/workflows/release.drafter.yml@main
    secrets: inherit
```

#### Inputs

Input | Type | Default | Required
--- | --- | --- | ---

#### Steps

1. on merge to `main`, updates Release draft

#### References
- learn more about github release management here: https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository
- Release drafter github action: https://github.com/release-drafter/release-drafter

---

### <a id="run.postman.api.tests"></a> run.postman.api.tests
Runs Postman API tests and outputs junit and html reports.

#### Usage
Entire worfklow for reference to add to individual repository.
```
name: Run Postman API Tests Workflow
on:
  push:
    branches: 'main'

jobs:
  run-postman-api-tests:
    uses: flexbase-eng/.github/.github/workflows/postman.api.test.yml@main
    with:
      postman_collection_directory: <your postman collection directory goes here>
      postman_environment_directory: <your post man environment directory goes here>
```

#### Inputs

Input | Type | Default | Required
--- | --- | --- | ---
postman_collection_directory | string | none | yes
postman_environment_directory | string | none | yes

#### Steps

1. Determine the directory for Postman collection where API tests are located
2. Determine the directory for Postman environment file needed to run the tests

# model-deployment-template

A template repository for the deployment of `spack`-based models.

> [!NOTE]
> Feel free to replace this README with information on the model once the TODOs have been ticked off.

## Things TODO to get your model deployed

### Settings

#### Repository Settings

Branch protections should be set up on `main` and the special `backport/*.*` branches, which are used for backporting of fixes to major releases (the `YEAR.MONTH` portion of the `YEAR.MONTH.MINOR` version) of models.

#### Repository Secrets/Variables

There are a few secrets and variables that must be set at the repository level.

##### Repository Secrets

* `BUILD_DB_CONNECTION_STR`: A postgresql connection url to the release provenance database
* `GH_COMMIT_CHECK_TOKEN`: GitHub Token that allows workflows to run based on workflow-authored commits (in  the case where a user uses `!bump` commands in PRs that bumps the version of the model)

##### Repository Variables

* `BUILD_DB_PACKAGES`: List of `spack` packages that are model components that will be uploaded to the release provenance database
* `NAME`: which corresponds to the model name - which is usually the repository name
* `CONFIG_VERSIONS_SCHEMA_VERSION`: Version of the [`config/versions.json` schema](https://github.com/ACCESS-NRI/schema/tree/main/au.org.access-nri/model/deployment/config/versions) used in this repository
* `SPACK_YAML_SCHEMA_VERSION`: Version of the [ACCESS-NRI-style `spack.yaml` schema](https://github.com/ACCESS-NRI/schema/tree/main/au.org.access-nri/model/spack/environment/deployment) used in this repository

#### Environment Secrets/Variables

GitHub Environments are sets of variables and secrets that are used specifically to deploy software, and hence have more security requirements for their use.

Currently, we have two Environments per deployment target - one for `Release` and one for `Prerelease`. Our current list of deployment targets and Environments can be found in this [deployment configuration file in `build-cd`](https://github.com/ACCESS-NRI/build-cd/blob/main/config/deployment-environment.json).

In order to deploy to a given deployment target:

* Environments with the name of the deployment target must be created _in this repository_ and have the associated secrets/variables set ([see below](#environment-secrets))
* There must be a `Prerelease` Environment associated with the `Release` Environment. For example, if we are deploying to `SUPERCOMPUTER`, we require Environments with the names `SUPERCOMPUTER`, `SUPERCOMPUTER Prerelease`.

When setting the environment up, remember to require sign off by a member of ACCESS-NRI when deploying as a `Release`.

Regarding the secrets and variables that must be created:

##### Environment Secrets

* `HOST`: The deployment location SSH Host
* `HOST_DATA`: The deployment location SSH Host for data transfer (may be the same as `HOST`)
* `SSH_KEY`: A SSH Key that allows access to the above `HOST`/`HOST_DATA`
* `USER`: A Username to login to the above `HOST`/`HOST_DATA`

##### Environment Variables

* `DEPLOYMENT_TARGET`: Name of the deployment target for logging purposes
* `SPACK_INSTALLS_ROOT_LOCATION`: Path to the directory that contains all versions of a deployment of `spack`. For example, if `/some/apps/spack` is the `SPACK_INSTALLS_ROOT_LOCATION`, that directory will contain directories like `0.20`, `0.21`, `0.22`, which in turn contain an install of `spack`, `spack-packages` and `spack-config`
* `SPACK_YAML_LOCATION`: Path to a directory that will contain the `spack.yaml` from this repository during deployment
* (Optional) `SPACK_INSTALL_PARALLEL_JOBS`: Explicit number of parallel jobs for the installation of the given model. Must be either of the form `--jobs N` or unset (for the default `--jobs 16`).

### File Modifications

#### In `.github/workflows`

* Reminder that these workflows use `vars.NAME` (as well as inherit the above environment secrets) and hence these must be set.
* If the name of the root SBD for the model (in [`spack-packages`](https://github.com/ACCESS-NRI/spack-packages/tree/main/packages)) is different from the model name (for example, `ACCESS-ESM1.5`s root SBD is `access-esm1p5`), you must uncomment and set the `jobs.[pr-ci|pr-comment].with.root-sbd` line to the appropriate SBD name.

#### In `config/versions.json`

* `.spack` must be given a version. For example, it will clone the associated `releases/VERSION` branch of `ACCESS-NRI/spack` if you give it `VERSION`.
* `.spack-packages` should also have a CalVer-compliant tag as the version. See the [associated repo](https://github.com/ACCESS-NRI/spack-packages/tags) for a list of available tags.
* `.spack-config` should also have a CalVer-compliant tag as the version. See the [associated repo](https://github.com/ACCESS-NRI/spack-config/tags) for a list of available tags.

#### In `spack.yaml`

There are a few TODOs for the `spack.yaml`:

* `spack.specs`: Set the root SBD as the only element of `spack.specs`. This must also have an `@git.YEAR.MONTH.MINOR` version as it is the version of the entire deployment (and indeed will be a tag in this repository).
* `spack.packages.*`: In this section, you can specify the versions and variants of dependencies. Note that the first element of the `spack.packages.*.require` must be only a version. Variants and other configuration can be done on subsequent lines.
* `spack.packages.all`: Can set configuration for all packages. For example, the compiler used, or the target architecture.
* `spack.modules.default.tcl.include`: List of package names that will be explicitly included and available to `module load`.
* `spack.modules.default.tcl.projections`: For included modules, you must set the name of the module to be the same as the `spack.packages.*.require[0]` version, without the `@git.`.

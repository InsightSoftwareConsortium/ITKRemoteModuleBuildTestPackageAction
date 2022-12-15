# ITKRemoteModuleBuildTestPackageAction

This project provides reusable GitHub Actions workflows to 
build, test, and package ITK external modules.

The Insight Toolkit (ITK) is an open-source, cross-platform system
that provides developers with an extensive suite of software tools
for image analysis.
More information is available on the [ITK website](https://itk.org/)
or at the [ITK GitHub homepage](https://github.com/insightSoftwareConsortium/ITK).

## Motivation

The ITK ecosystem is driven by community contributions in the form of external
modules that provide expanded functionality. Chapter 9 of the 
[ITK Software Guide](https://itk.org/ItkSoftwareGuide.pdf),
"How To Create A Module", details the process to extend ITK.

Continuous Integration (CI) is a best software practice wherein proposed
code changes are automatically built and tested for quality assurance.
The goal of `ITKRemoteModuleBuildTestPackageAction` is to minimize
the CI development burden for community members by providing
workflows for common building, testing, and packaging procedures.

In most cases community members can leverage `ITKRemoteModuleBuildTestPackageAction` in
their external module projects for several advantages:
- Build procedures are provided as GitHub Actions (GHA) reusable workflow `.yml` files
  which integrate directly with GHA runners. These can be invoked for a given
  external module in its GitHub repository with minimal code.
- Build procedure updates are centralized. Rather than replicating build step updates
  directly in a custom build procedure for each external module, developers may simply
  update workflow commit tags to include process updates in external module CI.
  For example, updating a runner platform version from `ubuntu-18.04` to `ubuntu-20.04`
  would be introduced to `ITKRemoteModuleBuildTestPackageAction` once and then consumed
  by a variety of external modules by simply updating the `ITKRemoteModuleBuildTestPackageAction`
  commit hash accordingly.

## Provided Workflows

Reusable workflow scripts are provided in the `.github/workflows` directory. Workflows
are organized by target distribution:
- The `build-test-cxx` workflow provides processes for building a module
  from its C++ source code, running module tests, and then reporting
  those test results to CDash. The module is built and tested on
  Windows, Ubuntu, and MacOS platforms with provided GHA runners.
  Results are visible at the [ITK CDash dashboard](https://open.cdash.org/index.php?project=Insight).
- The `build-test-package-python` workflow builds and packages external module
  Python wheels for Windows, Ubuntu, and MacOS platforms. The workflow uses
  build scripts provided at [ITKPythonPackage](https://github.com/InsightSoftwareConsortium/ITKPythonPackage).

## Usage

`ITKRemoteModuleBuildTestPackageAction` may be used by ITK external modules that are
hosted on GitHub to make use of GHA CI runners. The provided workflows assume that
any build dependencies other than ITK will be fetched during the CMake build process.

Reusable workflows may be referenced inside an external module's
`.github/workflows/build-test-package.yml` file to run when changes to the module are proposed.
Input parameters may also be passed to the reusable workflow to guide the build/test procedure.
See the `inputs:` field in any `ITKRemoteModuleBuildTestPackageAction` file for
a list of parameters available for use.

More information on GitHub Actions reusable workflows is available in
[GitHub documentation](https://docs.github.com/en/actions/using-workflows/reusing-workflows#calling-a-reusable-workflow).

## Example Usage

An example GHA `.yml` file using `ITKRemoteModuleBuildTestPackageAction` workflows:

```yaml
name: Build, test, package

on: [push,pull_request]

jobs:
  cxx-build-workflow:
    uses: InsightSoftwareConsortium/ITKRemoteModuleBuildTestPackageAction/.github/workflows/build-test-cxx.yml@d4a5ce4f219b66b78269a15392e15c95f90e7e00
    with:
      itk-cmake-options: '-DITK_BUILD_DEFAULT_MODULES:BOOL=OFF -DITKGroup_Core:BOOL=ON'

  python-build-workflow:
    uses: InsightSoftwareConsortium/ITKRemoteModuleBuildTestPackageAction/.github/workflows/build-test-package-python.yml@d4a5ce4f219b66b78269a15392e15c95f90e7e00
    secrets:
      pypi_password: ${{ secrets.pypi_password }}
```

The example above can be broken down line by line:

```yaml
name: Build, test, package
````
The workflow name that will be used to group workflow runs under
the "Action" tab on the external module's GitHub page.

```yaml
on: [push, pull_request]
```
Run workflows every time a new commit is pushed or a pull request is entered on the module repository.

```yaml
jobs:
```
The top-level jobs used to organize the run. Reusable workflows may provide multiple jobs.

```yaml
  cxx-build-workflow:
    uses: InsightSoftwareConsortium/ITKRemoteModuleBuildTestPackageAction/.github/workflows/build-test-cxx.yml@d4a5ce4f219b66b78269a15392e15c95f90e7e00
```
Tells GHA to fetch and run the `build-test-cxx.yml` workflow.
A commit hash or tagged version may be provided.

```yaml
    with:
      itk-cmake-options: '-DITK_BUILD_DEFAULT_MODULES:BOOL=OFF -DITKGroup_Core:BOOL=ON'
```
Passes input arguments to the reusable workflow. In this case the parameters provided
in `itk-cmake-options` will be passed to the ITK configuration command so that only
certain modules are built before the external module itself is subsequently built against ITK.

```yaml
  python-build-workflow:
    uses: InsightSoftwareConsortium/ITKRemoteModuleBuildTestPackageAction/.github/workflows/build-test-package-python.yml@d4a5ce4f219b66b78269a15392e15c95f90e7e00
```
Tells GHA to fetch and run the `build-test-package-python.yml` workflow.
A commit hash or tagged version may be provided.

```yaml
  secrets:
    pypi_password: ${{ secrets.pypi_password }}
```
Passes a secret from the external module repository to the workflow.
In this case the `pypi_password` secret is required for automatically uploading
Python wheel to the [Python Package Index (PyPI)](https://pypi.org/) for distribution.

ITKSplitComponents is one example of an external module leveraging reusable workflows for continuous integration.
Refer to [ITKSplitComponent's GHA workflow](https://github.com/InsightSoftwareConsortium/ITKSplitComponents/blob/master/.github/workflows/build-test-package.yml)

## Contributing

Community contributions to `ITKRemoteModuleBuildTestPackageAction` are welcome!

Please see [ITK's CONTRIBUTING.MD](https://github.com/InsightSoftwareConsortium/ITK/blob/master/CONTRIBUTING.md)
for general best practices regarding ITK and external modules.

Workflow issues may be submitted to the `ITKRemoteModuleBuildTestPackageAction`
[issue tracker](https://github.com/InsightSoftwareConsortium/ITKRemoteModuleBuildTestPackageAction/issues)
on GitHub.

## Additional Notes

`ITKRemoteModuleBuildTestPackageAction` aims to provide boilerplate to simplify CI processes
for the majority of ITK C++ external modules. However, it may not be feasible to adapt
reusable workflows to fit every project's needs, particularly in the case where
build setup requires fetching project-specific third party dependencies before building.

If your project requires build steps not currently addressed by `ITKRemoteModuleBuildTestPackageAction`
then consider doing one of the following:
1. Implement build setup steps in your project's CMake procedures in `CMakeLists.txt` such that
   building with CMake in a reusable workflow handles extraneous dependency retrieval steps.
2. Fork the workflows in `ITKRemoteModuleBuildTestPackageAction` for your project and manually
   replicate subsequent updates in your workflow;
3. Open a pull request with your proposed changes if the changes will generally benefit other external
   modules in the ITK ecosystem;
4. [Open an issue](https://github.com/InsightSoftwareConsortium/ITKRemoteModuleBuildTestPackageAction/issues/new)
   on `ITKRemoteModuleBuildTestPackageAction`.

## Community Discussion

Please direct general questions and discussions to the [ITK Discourse forum](https://discourse.itk.org/).

## License

`ITKRemoteModuleBuildTestPackageAction` is distributed under the Apache License 2.0 (see [LICENSE](LICENSE))

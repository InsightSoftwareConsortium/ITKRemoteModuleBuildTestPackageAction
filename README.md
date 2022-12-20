# ITKRemoteModuleBuildTestPackageAction

This project provides reusable GitHub Actions workflows to 
build, test, and package ITK external modules.

The Insight Toolkit (ITK) is an open-source, cross-platform system
that provides developers with an extensive suite of software tools
for image analysis.
More information is available on the [ITK website](https://itk.org/)
or at the [ITK GitHub homepage](https://github.com/insightSoftwareConsortium/ITK).

## Table of Contents

- [Motivation](#motivation)
- [Usage](#usage)
  - [Example Usage](#example-usage)
  - [`build-test-cxx` Overview](#build-test-cxx-overview)
  - [`build-test-package-python` Overview](#build-test-package-python-overview)
- [Contributing](#contributing)
- [Additional Notes](#additional-notes)
- [Community Discussion](#community-discussion)
- [License](#license)

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

### Example Usage

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

### `build-test-cxx` Overview

The workflow defined in `build-test-cxx.yml` builds and tests the external module
against a recent version of the Insight Toolkit.
For efficiency, wrappings are not typically built as part of this workflow.

Several optional parameters are exposed to allow the external module
to direct workflow execution.

- `itk-git-tag`: The git tag or commit hash to use to fetch ITK.

```yaml
  with:
    itk-git-tag: 'v5.3.0'
```

- `itk-cmake-options`: CMake configuration parameters for building ITK as a prerequisite.

```yaml
  with:
    itk-cmake-options: '-DITK_BUILD_DEFAULT_MODULES:BOOL=OFF'
```

- `itk-module-deps`: List of ITK remote module dependencies to build. Module name and
    git tag are passed to ITK to be fetched and built during the ITK build procedure.
    Note that the C++ build pipeline currently supports only modules that have
    a `.remote.cmake` entry under [ITK/Modules/Remote](https://github.com/InsightSoftwareConsortium/ITK/tree/master/Modules/Remote).
    The list is colon-delimited and must be in order of subsequent dependencies, i.e.
    if module `B` depends on module `A` then module `A` must be listed first.
    A git tag or commit hash must be provided for each module entry.
    The list is formatted as `module_A@tag:module_B@tag:...`.
    Note that the "ITK" prefix is omitted for fetching remote modules.

```yaml
  with:
    itk-module-deps: 'MeshToPolyData@3ad8f08:BSplineGradient@0.3.0'
```

- `cmake-options`: CMake configuration parameters for the module under test.

```yaml
  with:
    cmake-options: '-DBUILD_EXAMPLES:BOOL=ON'
```

- `warnings-to-ignore`: Warning string patterns that should be ignored for CTest/CDash reporting.
    If the given warning text is found in build output then it should neither be reported as a
    warning nor result in a build "failure". The input to this parameter should be escaped with
    double quotations and multiple warnings should be separated by spaces.

```yaml
  with:
    warnings-to-ignore: '\"libcleaver.a.*has no symbols\" \"libitkcleaver.*has no symbols\"'
```

- `ctest-options`: CTest command line interface (CLI) run options for the module under test.
    CTest options are described in [CTest documentation](https://cmake.org/cmake/help/latest/manual/ctest.1.html).
    This parameter can be used to disable tests that are known to be broken, increase
    output verbosity, and more. Some options may be always enabled by default.

```yaml
  with:
    ctest-options: '-E \"elastix_run_example_COMPARE_IM|elastix_run_3DCT_lung.MI.bspline.ASGD.001_COMPARE_TP\"'
```

### `build-test-package-python` Overview

The workflow defined in `build-test-package-python.yml` builds and packages Python
wrappings for the external module against a fixed version of the Insight Toolkit.

Several optional parameters are exposed to allow the external module
to direct workflow execution.

- `cmake-options`: CMake configuration parameters for the module under test.

```yaml
  with:
    cmake-options: '-DELASTIX_USE_OPENMP:BOOL=OFF'
```

- `itk-wheel-tag`: The [GitHub release](https://github.com/InsightSoftwareConsortium/ITKPythonBuilds/releases)
    version tag for the
    [ITKPythonBuilds](https://github.com/insightSoftwareConsortium/ITKpythonbuilds) archive to use.
    In order to efficiently package arbitrary ITK external modules within GitHub Actions runners time limits
    the Insight Software Consortium provides tagged ITK build caches at
    [ITKPythonBuilds](https://github.com/insightSoftwareConsortium/ITKpythonbuilds) that correspond to
    `pip`-installable [ITK releases on the Python Package Index](https://pypi.org/project/itk/).
    Setting this parameter will build the external module against a fixed ITK build cache
    corresponding to a certain ITK wheel release on PyPI. See
    [ITKPythonBuilds/releases](https://github.com/InsightSoftwareConsortium/ITKPythonBuilds/releases)
    for a list of available tags to use.

```yaml
  with:
    itk-wheel-tag: 'v5.3.0'
```

- `itk-python-package-tag`: The git tag or commit hash for ITKPythonPackage build scripts to use.
    The [ITKPythonPackage](https://github.com/InsightSoftwareConsortium/ITKPythonPackage) repository
    contains scripts for building ITK wheels and ITK external module wheels for different Python
    versions on MacOS, Linux, and Windows target platforms. Setting this parameter will direct the
    workflow to use a certain script revision for building Python wheels. See available
    tags at [ITKPythonPackage/releases](https://github.com/InsightSoftwareConsortium/ITKPythonPackage/tags).

```yaml
  with:
    itk-python-package-tag: 'v5.3.0'
```

- `itk-python-package-org`: The GitHub organization to reference for fetching ITKPythonPackage build scripts.
    The central script repository is maintained on GitHub at
    [InsightSoftwareConsortium/ITKPythonPackage](https://github.com/InsightSoftwareConsortium/ITKPythonPackage).
    Setting this parameter will direct the workflow to fetch from an ITKPythonPackage fork on a different
    GitHub user's or organization's account. _Using this parameter is discouraged for purposes other than
    testing ITKPythonPackage development branches._

```yaml
  with:
    itk-python-package-org: 'InsightSoftwareConsortium'
```

- `itk-module-deps`: List of ITK remote module dependencies to build. Module name and
    git tag are iteratively fetched and built during the ITK build procedure.
    Modules need not be listed under
    [ITK/Modules/Remote](https://github.com/InsightSoftwareConsortium/ITK/tree/master/Modules/Remote).
    The list is colon-delimited and must be in order of subsequent dependencies, i.e.
    if module `B` depends on module `A` then module `A` must be listed first.
    A git tag or commit hash must be provided for each module entry.
    The list is formatted as `org/module_A@tag:org/module_B@tag:...`.

```yaml
  with:
    itk-module-deps: 'InsightSoftwareConsortium/ITKMeshToPolyData@3ad8f08:InsightSoftwareConsortium/ITKBSplineGradient@0.3.0'
```

- `manylinux-platforms`: List of [manylinux](https://github.com/pypa/manylinux)
    specialization build targets for Linux Python module wheels. Manylinux "provides
    a convenient way to distribute binary Python extensions as wheels on Linux";
    see the [manylinux README](https://github.com/pypa/manylinux#manylinux) for more details.
    A list entry includes both the name of the manylinux specialization, which is
    related to the toolset to be used, and the platform architecture to target.
    All supported specialization and architecture pairs are enabled by default
    and listed in the example below, but can be disabled by removing them from the
    input list in the calling external module workflow.
    The list is colon-delimited and wheels will be built in the order provided.
    The list is formatted as "<specialization>-<target_arch>:<specialization>-<target_arch>".

```yaml
  with:
    manylinux-platforms: '_2_28-aarch64:_2_28-x64:2014-x64'
```

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
   replicate subsequent updates in your workflow.
3. Open a pull request to propose changes if the changes will generally benefit other external
   modules in the ITK ecosystem.
4. [Open an issue](https://github.com/InsightSoftwareConsortium/ITKRemoteModuleBuildTestPackageAction/issues/new)
   on `ITKRemoteModuleBuildTestPackageAction`.

## Community Discussion

Please direct general questions and discussions to the [ITK Discourse forum](https://discourse.itk.org/).

## License

`ITKRemoteModuleBuildTestPackageAction` is distributed under the Apache License 2.0 (see [LICENSE](LICENSE))

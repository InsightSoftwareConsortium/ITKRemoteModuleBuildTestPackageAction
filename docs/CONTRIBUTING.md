# Contributing to `ITKRemoteModuleBuildTestPackageAction`

The Insight Toolkit (ITK) open-source community is driven by
involvement from users and developers alike. Contributions
can include new features, bug fixes, or simply
documentation updates. This document describes the community
guidelines to help contributors (like you!) improve the
`ITKRemoteModuleBuildTestPackageAction` repository.

Check out the project [README](../README.md)
before reading this to better understand the purpose and components
of the `ITKRemoteModuleBuildTestPackageAction` repository.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Creating or Modifying a Workflow](#creating-or-modifying-a-workflow)
- [Testing Your Changes](#testing-your-changes)
- [Creating a Pull Request](#creating-a-pull-request)

## Prerequisites

In order to contribute to `ITKRemoteModuleBuildTestPackageAction` you
must have the following:
- A computer;
- An internet connection;
- A [GitHub](https://github.com/) account;
- A [git](https://git-scm.com/) client;
- A text editor of your choice.

The following are also helpful, but not strictly required:
- A computer running the operating system for which your changes are targeted,
  i.e. Windows, Ubuntu (Linux), or macOS. This will allow you to test changes locally
  before pushing to GitHub.
- A Python interpreter of 3.8 or greater for testing wheels locally.

Before getting started it's helpful to have a working understanding of the following:
- ITK external modules
  (see [README discussion](../README.md#motivation));
- [GitHub Actions](https://docs.github.com/en/actions) and
  [continuous integration](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration)
  in general;
- [GitHub Actions Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows).

The following tools and concepts are used in `ITKRemoteModuleBuildTestPackageAction`
and may be helpful or required for contributing:

- [Bash](https://www.gnu.org/software/bash/) and
  [PowerShell](https://learn.microsoft.com/en-us/powershell/) scripting;
- [CMake](https://cmake.org/) and [Ninja](https://ninja-build.org/) build tools;
- [CTest](https://cmake.org/cmake/help/latest/manual/ctest.1.html) and 
  [CDash](https://cmake.org/cmake/help/book/mastering-cmake/chapter/CDash.html)
  testing and reporting tools;
- [Python](https://www.python.org/) scripting;
- [Python wheel](https://wheel.readthedocs.io/en/stable/) packaging;

## Creating or Modifying a Workflow

GitHub Actions requires that workflows are placed in the [`.github/workflows`](../.github/workflows) directory.
This is where you will find existing workflows and possibly add new workflows.

To contribute to `ITKRemoteModuleBuildTestPackageAction` you will need to
first [fork](https://docs.github.com/en/github-ae@latest/get-started/quickstart/fork-a-repo)
the repository to your user account on GitHub and then
[clone](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository)
the fork to your local machine.

### Creating a Workflow

`ITKRemoteModuleBuildTestPackageAction` is a central repository for GitHub Actions
reusable workflows for ITK external modules. Before creating a workflow, ask yourself
the following questions:
1. Does my workflow address a problem for ITK external modules that is not
   already solved?
1. Does my workflow provide functionality that will generalize to various ITK external modules?
1. Does my workflow provide functionality related to continuous integration such as
   building, packaging, and/or testing steps?

If the answer to all of the above is "yes", then continue on with your new workflow!

Reference
[GitHub Actions Reusable Workflow documentation](https://docs.github.com/en/actions/using-workflows/reusing-workflows#creating-a-reusable-workflow)
to get started. You will need to:
1. Create a workflow in the `.github/workflows` folder;
1. Define workflow inputs;
1. Define how jobs will be launched;
1. Define steps that a given job will take.

### Modifying a Workflow

Existing workflows will need to be maintained and updated over time as tools
and technology progress. Workflow modifications will typically be accepted
only if:
1. The modification addresses an existing, documented issue in the reusable
   workflow that prevents the workflow from operating as intended; or
1. The modification adds a minor feature update that is deemed suitable
   for integration with an existing workflow rather than the creation
   of a new workflow. The feature update will benefit an assortment
   of ITK external modules.

The best approach to modifying a workflow is to recreate steps on your local 
system (if possible), identify failures and fixes, then translate fixes to the
reusable workflow in question. If you encounter an issue that requires
community input you can ask for help on the
[ITK community Discourse forum](https://discourse.itk.org/).

## Testing Your Changes

Reusable workflows are intended to _manage continuous integration_ for
ITK external modules in a reusable manner. As of January 2023
there are unfortunately no good options for establishing continuous
integration around proposed changes to reusable workflows themselves.
It is therefore necessary to place an extra burden on the contributors
to `ITKRemoteModuleBuildTestPackageAction` to schedule tests themselves.
Fortunately, this severely reduces the overhead required of the
general ITK community in maintaining continuous integration on individual
external modules.

Changes in reusable workflows should be tested through application to
ITK external modules in a user fork. The ideal repositories to use
for testing will depend on the workflow changes under test.

The [ITKSplitComponents repository](https://github.com/InsightSoftwareConsortium/ITKSplitComponents)
is a good target for testing relatively simple changes. `ITKSplitComponents`
is a small header-only external module with a minimum testing suite,
Python wrappings, and an example Jupyter Notebook.
Do the following to test against `ITKSplitComponents` or another ITK external module:
1. Commit your changes to `ITKRemoteModuleBuildTestPackageAction` and
   push to make them available on your GitHub user account. See the
   [ITK "Contributing" documentation](https://github.com/InsightSoftwareConsortium/ITK/blob/master/CONTRIBUTING.md#commit-messages)
   for guidelines on writing commit messages in the ITK ecosystem.
```sh
$ git commit
$ git push
```

2. Fork the [ITKSplitComponents repository](https://github.com/InsightSoftwareConsortium/ITKSplitComponents)
    to your GitHub user account and clone to your local machine:
```sh
$ cd ..
$ git clone git@github.com:<your-username>/ITKSplitComponents.git
$ cd ITKSplitComponents
```

3. Create a development branch for your proposed changes:
```sh
$ git checkout -b "my-cool-feature"
```

4. Update the external module workflow `.yml` file to reference your changes. Either the name of the
development branch or the desired development commit hash may be used to reference workflows.
```yml
name: Build, test, package

on: [push,pull_request]

jobs:
  cxx-build-workflow:
    uses: <your-username>/ITKRemoteModuleBuildTestPackageAction/.github/workflows/build-test-cxx.yml@<your-dev-branch>
    with:
      itk-cmake-options: '-DITK_BUILD_DEFAULT_MODULES:BOOL=OFF -DITKGroup_Core:BOOL=ON'

  python-build-workflow:
    uses: <your-username>/ITKRemoteModuleBuildTestPackageAction/.github/workflows/build-test-package-python.yml@<your-dev-branch>
    secrets:
      pypi_password: ${{ secrets.pypi_password }}
```

5. Commit and push your changes:
```sh
$ git commit
$ git push
```

6. Visit `https://github.com/<your-username>/ITKSplitComponents/actions` to view running Github Actions jobs.

Other ITK external modules may also be used for testing:
- [ITKBSplineGradient](https://github.com/InsightSoftwareConsortium/ITKBSplineGradient) can be
  used for building with a dependency on one other ITK external module;
- [ITKUltrasound](https://github.com/KitwareMedical/ITKUltrasound) can be used for building with
  dependencies on multiple ITK external modules.

## Creating a Pull Request

You can propose changes to the central `InsightSoftwareConsortium` repository by
creating a
[pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request).

1. Visit `https://github.com/<your-username>/ITKRemoteModuleBuildTestPackageAction/pulls` and
   click the green "New Pull Request" button to create a pull request for merging changes
   from your development branch into `InsightSoftwareConsortium/main`.
1. Describe your changes and create the pull request.

Each nontrivial `ITKRemoteModuleBuildTestPackageAction` pull request _must_ include a
link to a successful GitHub Actions test run in its description. Refer to the
[Testing Your Changes](#testing-your-changes) section for instructions on how to
set up a test run in a user fork of an ITK external module.

After a repository maintainer reviews, approves, and merges your pull request
your changes will be available to all of the ITK external modules that use
`ITKRemoteModuleBuildTestPackageAction` reusable workflows to drive their testing.
Modules typically use a tagged version of reusable workflows to minimize disruption,
so the changes will be applied to each module when it updates to the workflow
commit hash that includes your changes.

Thank you for contributing to `ITKRemoteModuleBuildTestPackageAction`!

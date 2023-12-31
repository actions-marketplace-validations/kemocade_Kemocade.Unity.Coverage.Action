# Kemocade Unity Coverage Action
A github action for running unity test coverage quality gate.
This github action parses the output of the [unity code coverage package](https://docs.unity3d.com/Packages/com.unity.testtools.codecoverage@0.2/manual/CoverageTestRunner.html) and fails the action/github pull requests, if code coverage requirements are not met.

Forked from [UnityCodeCoverage.Action](https://github.com/ActuatorDigital/UnityCodeCoverage.Action).
Updated to .NET 7 & C# 11.0 by [Dustuu](https://github.com/dustuu) from [Kemocade](https://github.com/kemocade).

## Inputs

### `coverage-file-path` - **Required** 
The path of the coverage file to parse.

Default: './artifacts/CodeCoverage/Report/Summary.xml'

### `required-coverage` - **Required**
The minimum percentage of code coverage requried for the action to sucessfully complete. 

Default: 75

## Example usage
First, if you haven't already, set up a new build action in your project's /.github/workflows/main.yml. 
```
name: Unity CI 

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ develop, master, release ]
  pull_request:
    branches: [ develop, master, release ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:

```
This Unity code coverage action dependes on artifacts from the test job, set that job to run first. Great documentation on how to setup and activate unity inside an action container has been made available by the [untiy-actions repo author webbertakken](https://github.com/webbertakken/unity-actions).

```
  test:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - name: Clone Repo
      uses: actions/checkout@v2

    # Cache unity Library folder from previous builds.
    - name: Restore Unity Library Folder
      uses: actions/cache@v1.1.0
      with:
        path: ./Library
        key: Library-Hex-UNITY_STANDALONE_LINUX
        restore-keys:  |
          Library-Hex-
    
    # Test
    - name: Test Runner
      uses: game-ci/unity-test-runner@v3.0.0
      env:
        UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}
        UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }}
        UNITY_SERIAL: ${{ secrets.UNITY_SERIAL }}
      with:
        githubToken: ${{ secrets.GITHUB_TOKEN }}
        projectPath: ./
        unityVersion: 2019.3.4f1

    - name: Save Test Result Artifacts 
      uses: actions/upload-artifact@v1
      with:
          name: Test results
          path: artifacts
```
Once the tests job has run, and the artifacts have been uploaded, a job for this repo's action can be run to assert code coverage requirements are met.
```
  code-coverage:
      # The type of runner that the job will run on
      name: coverage
      needs: test
      runs-on: ubuntu-latest
      steps:
        - name: Load Test Result Artifacts 
          uses: actions/download-artifact@v1
          with:
              name: Test results
              path: artifacts
        - shell: bash
          run: cat ./artifacts/CodeCoverage/Report/Summary.xml
      
        # Ensure code coverage exceeds required coverage.
        - name: Check Code Coverage
          uses: kemocade/Kemocade.Unity.Coverage.Action@1.0.0
          with:
            coverage-file-path: ./artifacts/CodeCoverage/Report/Summary.xml
            required-coverage: 25

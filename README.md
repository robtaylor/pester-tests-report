# pester-tests-report

GitHub Action to run Pester tests and generate report attached to Workflow Run


dfwef
---

[![GitHub Workflow - CI](https://github.com/zyborg/pester-tests-report/workflows/test-action/badge.svg)](https://github.com/zyborg/pester-tests-report/actions?workflow=test-action)
[![GitHub release (latest by date)](https://img.shields.io/github/v/release/zyborg/pester-tests-report)](https://github.com/zyborg/pester-tests-report/releases/latest)

---

This Action can be used to execute PowerShell Pester tests within a GitHub
Workflow, as well as generate a Report from the test results and attach it
to the Workflow Run as a Check Run.

Check out the [usage](#usage) below.

## Samples

Here we see some badges generated along with links to the _Gist-based_
Tests Report as part of a GitHub Workflow
[associated](https://github.com/zyborg/pester-tests-report/actions/runs/298169540/workflow#L138)
with this project.

* [![Example Pester Tests Badge](https://gist.github.com/ebekker/9158b8c9a9999270ef482faae6f4a742/raw/pester-tests-report_Marketplace_GHAction_test1.md_badge.svg)](https://gist.github.com/ebekker/9158b8c9a9999270ef482faae6f4a742#pester-tests-report_Marketplace_GHAction_test1.md)
* [![Example Pester Tests Badge](https://gist.github.com/ebekker/5e97f7385f1c7812118d39306b938cd0/raw/pester-tests-report_Marketplace_GHAction_test2.md_badge.svg)](https://gist.github.com/ebekker/5e97f7385f1c7812118d39306b938cd0#pester-tests-report_Marketplace_GHAction_test2.md)
* [![Example Pester Tests Badge](https://gist.github.com/ebekker/dff5ae43943226d800cd1ee891dc889b/raw/pester-tests-report_GHAction_test1.md_badge.svg)](https://gist.github.com/ebekker/dff5ae43943226d800cd1ee891dc889b)

And here are some samples of the actual generated reports:

<table border="2">
    <tr><td><img src="docs/sample1.png" /></td></tr>
</table><table border="2">
    <tr><td><img src="docs/sample2.png" /></td></tr>
</table><table border="2">
    <tr><td><img src="docs/sample3.png" /></td></tr>
</table>

## Usage

This Action allows you to specify a combination of filter inputs to resolve
tests and invoke [Pester](https://github.com/pester/Pester) on them, and then
generate a markdown report from the test results.

Alternatively, if you would like complete control of the testing process, you
can just generate your own test results report in the
[NUnit Test Results 2.5](https://docs.nunit.org/articles/nunit/technical-notes/usage/XML-Formats.html#v2-test-results)
format and simply have this Action produce the markdown report.

Here's a quick example of how to use this action in your own GitHub Workflows.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: test module
        id: test_module
        uses: zyborg/pester-tests-report@v1
        with:
          include_paths: tests
          exclude_paths: tests/powershell1,tests/powershell2
          exclude_tags: skip_ci
          report_name: module_tests
          report_title: My Module Tests
          github_token: ${{ secrets.GITHUB_TOKEN }}
    - name: dump test results
      shell: pwsh
      run: |
        Write-Host 'Total Tests Executed...:  ${{ steps.test_module.outputs.total_count }}'
        Write-Host 'Total Tests PASSED.....:  ${{ steps.test_module.outputs.passed_count }}'
        Write-Host 'Total Tests FAILED.....:  ${{ steps.test_module.outputs.failed_count }}'
```

Additionally (or alternatively) you can generate a Gist-based report, and have it updated
with each build.  You can also generate a test result badge along with the Gist-based report
for reference such as on a README page.  Here's an example badge:

[![Example Pester Tests Badge](https://gist.github.com/ebekker/dff5ae43943226d800cd1ee891dc889b/raw/pester-tests-report_GHAction_test1.md_badge.svg)](https://gist.github.com/ebekker/dff5ae43943226d800cd1ee891dc889b)

### Inputs

This Action defines the following formal inputs.

| Name | Req | Description
|-|-|-|
| **`test_results_path`**  | false | Path to the test results file which will be used to generate a report.  If this path is provided, it will override the execution of all Pester tests, so the **`include_paths`** input and all other inputs that drive the test invocation behavior will be ignored. Instead a report will be generated based on the results stored in the file pointed to by this path. At this time, the only test results format supported is the NUnit 2.5 Tests results XML schema.
| **`full_names_filters`** | false | Comma-separated list of test full names, or with `-like` wildcards, to restrict which tests are resolved.  The default is to include every test discovered matching the other input parameters.
| **`include_paths`**      | false | Comma-separated list of one or more test files or directories that contain test files, relative to the root of the project. This path is optional and defaults to the project root, in which case all tests script across the entire project tree will be discovered.  A test script is identified by the filename pattern `*.Tests.ps1`. This parameter is _ignored_ if the **`test_results_path`** parameter is also provided.
| **`exclude_paths`**      | false | Comma-separated list of one or more test files or directories that contain test files, relative to the root of the project, that will be excluded from the list fo auto-discovered tests. This parameter is _ignored_ if the **`test_results_path`** parameter is also provided.
| **`include_tags`**       | false | Comma-separated list of tags to include for testing. This parameter is _ignored_ if the **`test_results_path`** parameter is also provided.
| **`exclude_tags`**       | false | Comma-separated list of tags to exclude for testing. This parameter is _ignored_ if the **`test_results_path`** parameter is also provided.
| **`output_level`**       | false | Optionally specify the level of output detail for the test results. May be one of: `none`, `Minimal`, `normal`, `detailed`, `diagnostic` The default is `normal`.
| **`report_name`**        | false | The name of the report object that will be attached to the Workflow Run.  Defaults to the name `TEST_RESULTS_<datetime>` where `<datetime>` is in the form `yyyyMMdd_hhmmss`.
| **`report_title`**       | false | The title of the report that will be embedded in the report itself, which defaults to the same as the `report_name` input.
| **`github_token`**       | true  | GITHUB_TOKEN to authenticate against API calls to attach report to Workflow Run.
| **`skip_check_run`**     | false | If true, will skip attaching the Tests Result report to the Workflow Run using a Check Run.  Useful if you just want to produce a Gist-based report via the `gist_name` and `gist_token` input parameters.
| **`gist_name`**          | false | If this value is specifed, the Test Results Report will be attached as a version of a Gist under the name of this input. The `gist_token` input is also required to use this feature.
| **`gist_badge_label`**   | false | If specified, the Test Report Gist will also include an adjacent badge rendered with the status of the associated Test Report and and label content of this input.  In addition to any static text you can provide _escape tokens_ of the form `%name%` where name can be the name of any field returned from a Pester Result, such as `ExecutedAt` or `Result`.  If you want a literal percent, just specify an empty name as in `%%`.
| **`gist_badge_message`** | false | If Gist badge generation is enabled by providing a value for the `gist_badge_label` input, this input allows you to override the default message on the badge, which is equivalent to the the Pester Result `Status` such as `Failed` or `Passed`.  As with the label input, you can specify escape tokens in addition to literal text.  See the label input description for more details.
| **`gist_token`**         | false | GitHub OAuth/PAT token to be used for accessing Gist to store test results report. The integrated GITHUB_TOKEN that is normally accessible during a Workflow does not include read/write permissions to associated Gists, therefore a separate token is needed. You can control which account is used to actually store the state by generating a token associated with the target account.



### Outputs

This Action defines the following formal outputs.

| Name | Description
|-|-|
| **`test_results_path`** | Path to the test results file.  If the same-named input was provided to this action, this value will be the same. Otherwise, this will be the path to where the test results file was generated from running the resolved Pester tests.
| **`error_message`** | The summary message of the error returned by Pester, or empty if no error was produced.
| **`error_clixml_path`** | If Pester produced an error during invocation, this will be the path to an export of the full error record in the CLIXML form.  A subsequent PowerShell step can recover this object using the `Import-Clixml` cmdlet.
| **`result_clixml_path`** | If Pester produced an invocation result, this will be the path to an export of the full result record in the CLIXML form.  A subsequent PowerShell step can recover this object using the `Import-Clixml` cmdlet.
| **`result_value`** | A single string indicating the final result such as `Failed` or `Passed`.
| **`total_count`** | Total number of tests discovered.
| **`passed_count`** | Total number of tests passed.
| **`failed_count`** | Total number of tests failed.

### PowerShell GitHub Action

This Action is implemented as a [PowerShell GitHub Action](https://github.com/ebekker/pwsh-github-action-base).

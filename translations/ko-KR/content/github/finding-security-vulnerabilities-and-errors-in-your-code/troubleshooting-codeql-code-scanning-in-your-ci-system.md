---
title: Troubleshooting CodeQL code scanning in your CI system
shortTitle: Troubleshooting in your CI
intro: 'If you''re having problems with the {% data variables.product.prodname_codeql_runner %}, you can troubleshoot by using these tips.'
product: '{% data reusables.gated-features.code-scanning %}'
redirect_from:
  - /github/finding-security-vulnerabilities-and-errors-in-your-code/troubleshooting-code-scanning-in-your-ci-system
versions:
  free-pro-team: '*'
  enterprise-server: '>=2.22'
---

{% data reusables.code-scanning.beta-codeql-runner %}
{% data reusables.code-scanning.beta %}

### The `init` command takes too long

Before the {% data variables.product.prodname_codeql_runner %} can build and analyze code, it needs access to the {% data variables.product.prodname_codeql %} bundle, which contains the {% data variables.product.prodname_codeql %} CLI and the {% data variables.product.prodname_codeql %} libraries.

When you use the {% data variables.product.prodname_codeql_runner %} for the first time on your machine, the `init` command downloads the {% data variables.product.prodname_codeql %} bundle to your machine. This download can take a few minutes. The {% data variables.product.prodname_codeql %} bundle is cached between runs, so if you use the {% data variables.product.prodname_codeql_runner %} again on the same machine, it won't download the {% data variables.product.prodname_codeql %} bundle again.

To avoid this automatic download, you can manually download the {% data variables.product.prodname_codeql %} bundle to your machine and specify the path using the `--codeql-path` flag of the `init` command.

### No code found during the build

If the `analyze` command for the {% data variables.product.prodname_codeql_runner %} fails with an error `No source code was seen during the build`, this indicates that {% data variables.product.prodname_codeql %} was unable to monitor your code. Several reasons can explain such a failure.

1. Automatic language detection identified a supported language, but there is no analyzable code of that language in the repository. A typical example is when our language detection service finds a file associated with a particular programming language like a `.h`, or `.gyp` file, but no corresponding executable code is present in the repository. To solve the problem, you can manually define the languages you want to analyze by using the `--languages` flag of the `init` command. For more information, see "[Configuring {% data variables.product.prodname_code_scanning %} in your CI system](/github/finding-security-vulnerabilities-and-errors-in-your-code/configuring-codeql-code-scanning-in-your-ci-system)."

1. You're analyzing a compiled language without using the `autobuild` command and you run the build steps yourself after the `init` step. For the build to work, you must set up the environment such that the {% data variables.product.prodname_codeql_runner %} can monitor the code. The `init` command generates instructions for how to export the required environment variables, so you can copy and run the script after you've run the `init` command.
   - On macOS and Linux:
     ```shell
      $ . codeql-runner/codeql-env.sh
     ```
   - On Windows, using the Command shell (`cmd`) or a batch file (`.bat`):
     ```shell
     > call codeql-runner\codeql-env.bat
     ```
   - On Windows, using PowerShell:
     ```shell
     > cat codeql-runner\codeql-env.sh | Invoke-Expression
     ```

   The environment variables are also stored in the file `codeql-runner/codeql-env.json`. This file contains a single JSON object which maps environment variable keys to values. If you can't run the script generated by the `init` command, then you can use the data in JSON format instead.

   {% note %}

   **Note:** If you used the `--temp-dir` flag of the `init` command to specify a custom directory for temporary files, the path to the `codeql-env` files might be different.

   {% endnote %}

1. The code is built in a container or on a separate machine. If you use a containerized build or if you outsource the build to another machine, make sure to run the {% data variables.product.prodname_codeql_runner %} in the container or on the machine where your build task takes place.
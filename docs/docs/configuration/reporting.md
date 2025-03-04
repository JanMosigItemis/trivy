# Reporting

## Supported Formats
Trivy supports the following formats:

- Table
- JSON
- [SARIF](https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning)
- Template
- SBOM

### Table (Default)

|     Scanner      | Supported |
|:----------------:|:---------:|
|  Vulnerability   |     ✓     |
| Misconfiguration |     ✓     |
|      Secret      |     ✓     |
|     License      |     ✓     |

```
$ trivy image -f table golang:1.12-alpine
```

#### Show origins of vulnerable dependencies

|     Scanner      | Supported |
|:----------------:|:---------:|
|  Vulnerability   |     ✓     |
| Misconfiguration |           |
|      Secret      |           |
|     License      |           |

!!! warning "EXPERIMENTAL"
    This feature might change without preserving backwards compatibility.

Modern software development relies on the use of third-party libraries.
Third-party dependencies also depend on others so a list of dependencies can be represented as a dependency graph.
In some cases, vulnerable dependencies are not linked directly, and it requires analyses of the tree.
To make this task simpler Trivy can show a dependency origin tree with the `--dependency-tree` flag.
This flag is only available with the `--format table` flag.

The following packages/languages are currently supported:

- OS packages 
    - apk
    - dpkg
    - rpm
- Node.js 
    - npm: package-lock.json
    - pnpm: pnpm-lock.yaml
    - yarn: yarn.lock
- .NET
    - NuGet: packages.lock.json
- Python 
    - Poetry: poetry.lock
- Ruby
    - Bundler: Gemfile.lock
- Rust
    - Binaries built with [cargo-auditable][cargo-auditable]
- Go
    - Modules: go.mod
- PHP
    - Composer

This tree is the reverse of the npm list command.
However, if you want to resolve a vulnerability in a particular indirect dependency, the reversed tree is useful to know where that dependency comes from and identify which package you actually need to update.

In table output, it looks like:

```sh
$ trivy fs --severity HIGH,CRITICAL --dependency-tree /path/to/your_node_project

package-lock.json (npm)
=======================
Total: 2 (HIGH: 1, CRITICAL: 1)

┌──────────────────┬────────────────┬──────────┬───────────────────┬───────────────┬────────────────────────────────────────────────────────────┐
│     Library      │ Vulnerability  │ Severity │ Installed Version │ Fixed Version │                           Title                            │
├──────────────────┼────────────────┼──────────┼───────────────────┼───────────────┼────────────────────────────────────────────────────────────┤
│ follow-redirects │ CVE-2022-0155  │ HIGH     │ 1.14.6            │ 1.14.7        │ follow-redirects: Exposure of Private Personal Information │
│                  │                │          │                   │               │ to an Unauthorized Actor                                   │
│                  │                │          │                   │               │ https://avd.aquasec.com/nvd/cve-2022-0155                  │
├──────────────────┼────────────────┼──────────┼───────────────────┼───────────────┼────────────────────────────────────────────────────────────┤
│ glob-parent      │ CVE-2020-28469 │ CRITICAL │ 3.1.0             │ 5.1.2         │ nodejs-glob-parent: Regular expression denial of service   │
│                  │                │          │                   │               │ https://avd.aquasec.com/nvd/cve-2020-28469                 │
└──────────────────┴────────────────┴──────────┴───────────────────┴───────────────┴────────────────────────────────────────────────────────────┘

Dependency Origin Tree (Reversed)
=================================
package-lock.json
├── follow-redirects@1.14.6, (HIGH: 1, CRITICAL: 0)
│   └── axios@0.21.4
└── glob-parent@3.1.0, (HIGH: 0, CRITICAL: 1)
    └── chokidar@2.1.8
        └── watchpack-chokidar2@2.0.1
            └── watchpack@1.7.5
                └── webpack@4.46.0
                    └── cra-append-sw@2.7.0
```

Vulnerable dependencies are shown in the top level of the tree.
Lower levels show how those vulnerabilities are introduced.
In the example above **axios@0.21.4** included in the project directly depends on the vulnerable **follow-redirects@1.14.6**.
Also, **glob-parent@3.1.0** with some vulnerabilities is included through chain of dependencies that is added by **cra-append-sw@2.7.0**.

Then, you can try to update **axios@0.21.4** and **cra-append-sw@2.7.0** to resolve vulnerabilities in **follow-redirects@1.14.6** and **glob-parent@3.1.0**.

### JSON

|     Scanner      | Supported |
|:----------------:|:---------:|
|  Vulnerability   |     ✓     |
| Misconfiguration |     ✓     |
|      Secret      |     ✓     |
|     License      |     ✓     |

```
$ trivy image -f json -o results.json golang:1.12-alpine
```

<details>
<summary>Result</summary>

```
2019-05-16T01:46:31.777+0900    INFO    Updating vulnerability database...
2019-05-16T01:47:03.007+0900    INFO    Detecting Alpine vulnerabilities...
```

</details>

<details>
<summary>JSON</summary>

```
[
  {
    "Target": "php-app/composer.lock",
    "Vulnerabilities": null
  },
  {
    "Target": "node-app/package-lock.json",
    "Vulnerabilities": [
      {
        "VulnerabilityID": "CVE-2018-16487",
        "PkgName": "lodash",
        "InstalledVersion": "4.17.4",
        "FixedVersion": "\u003e=4.17.11",
        "Title": "lodash: Prototype pollution in utilities function",
        "Description": "A prototype pollution vulnerability was found in lodash \u003c4.17.11 where the functions merge, mergeWith, and defaultsDeep can be tricked into adding or modifying properties of Object.prototype.",
        "Severity": "HIGH",
        "References": [
          "https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-16487",
        ]
      }
    ]
  },
  {
    "Target": "trivy-ci-test (alpine 3.7.1)",
    "Vulnerabilities": [
      {
        "VulnerabilityID": "CVE-2018-16840",
        "PkgName": "curl",
        "InstalledVersion": "7.61.0-r0",
        "FixedVersion": "7.61.1-r1",
        "Title": "curl: Use-after-free when closing \"easy\" handle in Curl_close()",
        "Description": "A heap use-after-free flaw was found in curl versions from 7.59.0 through 7.61.1 in the code related to closing an easy handle. ",
        "Severity": "HIGH",
        "References": [
          "https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-16840",
        ]
      },
      {
        "VulnerabilityID": "CVE-2019-3822",
        "PkgName": "curl",
        "InstalledVersion": "7.61.0-r0",
        "FixedVersion": "7.61.1-r2",
        "Title": "curl: NTLMv2 type-3 header stack buffer overflow",
        "Description": "libcurl versions from 7.36.0 to before 7.64.0 are vulnerable to a stack-based buffer overflow. ",
        "Severity": "HIGH",
        "References": [
          "https://curl.haxx.se/docs/CVE-2019-3822.html",
          "https://lists.apache.org/thread.html/8338a0f605bdbb3a6098bb76f666a95fc2b2f53f37fa1ecc89f1146f@%3Cdevnull.infra.apache.org%3E"
        ]
      },
      {
        "VulnerabilityID": "CVE-2018-16839",
        "PkgName": "curl",
        "InstalledVersion": "7.61.0-r0",
        "FixedVersion": "7.61.1-r1",
        "Title": "curl: Integer overflow leading to heap-based buffer overflow in Curl_sasl_create_plain_message()",
        "Description": "Curl versions 7.33.0 through 7.61.1 are vulnerable to a buffer overrun in the SASL authentication code that may lead to denial of service.",
        "Severity": "HIGH",
        "References": [
          "https://github.com/curl/curl/commit/f3a24d7916b9173c69a3e0ee790102993833d6c5",
        ]
      },
      {
        "VulnerabilityID": "CVE-2018-19486",
        "PkgName": "git",
        "InstalledVersion": "2.15.2-r0",
        "FixedVersion": "2.15.3-r0",
        "Title": "git: Improper handling of PATH allows for commands to be executed from the current directory",
        "Description": "Git before 2.19.2 on Linux and UNIX executes commands from the current working directory (as if '.' were at the end of $PATH) in certain cases involving the run_command() API and run-command.c, because there was a dangerous change from execvp to execv during 2017.",
        "Severity": "HIGH",
        "References": [
          "https://usn.ubuntu.com/3829-1/",
        ]
      },
      {
        "VulnerabilityID": "CVE-2018-17456",
        "PkgName": "git",
        "InstalledVersion": "2.15.2-r0",
        "FixedVersion": "2.15.3-r0",
        "Title": "git: arbitrary code execution via .gitmodules",
        "Description": "Git before 2.14.5, 2.15.x before 2.15.3, 2.16.x before 2.16.5, 2.17.x before 2.17.2, 2.18.x before 2.18.1, and 2.19.x before 2.19.1 allows remote code execution during processing of a recursive \"git clone\" of a superproject if a .gitmodules file has a URL field beginning with a '-' character.",
        "Severity": "HIGH",
        "References": [
          "http://www.securitytracker.com/id/1041811",
        ]
      }
    ]
  },
  {
    "Target": "python-app/Pipfile.lock",
    "Vulnerabilities": null
  },
  {
    "Target": "ruby-app/Gemfile.lock",
    "Vulnerabilities": null
  },
  {
    "Target": "rust-app/Cargo.lock",
    "Vulnerabilities": null
  }
]
```

</details>

`VulnerabilityID`, `PkgName`, `InstalledVersion`, and `Severity` in `Vulnerabilities` are always filled with values, but other fields might be empty.

### SARIF
|     Scanner      | Supported |
|:----------------:|:---------:|
|  Vulnerability   |     ✓     |
| Misconfiguration |     ✓     |
|      Secret      |     ✓     |
|     License      |     ✓     |

[SARIF][sarif] can be generated with the `--format sarif` flag.

```
$ trivy image --format sarif -o report.sarif  golang:1.12-alpine
```

This SARIF file can be uploaded to GitHub code scanning results, and there is a [Trivy GitHub Action][action] for automating this process.

### Template

|     Scanner      | Supported |
|:----------------:|:---------:|
|  Vulnerability   |     ✓     |
| Misconfiguration |     ✓     |
|      Secret      |     ✓     |
|     License      |     ✓     |

#### Custom Template

{% raw %}
```
$ trivy image --format template --template "{{ range . }} {{ .Target }} {{ end }}" golang:1.12-alpine
```
{% endraw %}

<details>
<summary>Result</summary>

```
2020-01-02T18:02:32.856+0100    INFO    Detecting Alpine vulnerabilities...
 golang:1.12-alpine (alpine 3.10.2)
```
</details>

You can compute different figures within the template using [sprig][sprig] functions.
As an example you can summarize the different classes of issues:


{% raw %}
```
$ trivy image --format template --template '{{- $critical := 0 }}{{- $high := 0 }}{{- range . }}{{- range .Vulnerabilities }}{{- if  eq .Severity "CRITICAL" }}{{- $critical = add $critical 1 }}{{- end }}{{- if  eq .Severity "HIGH" }}{{- $high = add $high 1 }}{{- end }}{{- end }}{{- end }}Critical: {{ $critical }}, High: {{ $high }}' golang:1.12-alpine
```
{% endraw %}

<details>
<summary>Result</summary>

```
Critical: 0, High: 2
```
</details>

For other features of sprig, see the official [sprig][sprig] documentation.

#### Load templates from a file
You can load templates from a file prefixing the template path with an @.

```
$ trivy image --format template --template "@/path/to/template" golang:1.12-alpine
```

#### Default Templates

If Trivy is installed using rpm then default templates can be found at `/usr/local/share/trivy/templates`.

##### JUnit
|     Scanner      | Supported |
|:----------------:|:---------:|
|  Vulnerability   |     ✓     |
| Misconfiguration |     ✓     |
|      Secret      |           |
|     License      |           |

In the following example using the template `junit.tpl` XML can be generated.
```
$ trivy image --format template --template "@contrib/junit.tpl" -o junit-report.xml  golang:1.12-alpine
```

##### ASFF
|     Scanner      | Supported |
|:----------------:|:---------:|
|  Vulnerability   |     ✓     |
| Misconfiguration |     ✓     |
|      Secret      |     ✓     |
|     License      |           |

Trivy also supports an [ASFF template for reporting findings to AWS Security Hub][asff]

##### HTML
|     Scanner      | Supported |
|:----------------:|:---------:|
|  Vulnerability   |     ✓     |
| Misconfiguration |     ✓     |
|      Secret      |           |
|     License      |           |

```
$ trivy image --format template --template "@contrib/html.tpl" -o report.html golang:1.12-alpine
```

The following example shows use of default HTML template when Trivy is installed using rpm.

```
$ trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o report.html golang:1.12-alpine
```

### SBOM
See [here](../supply-chain/sbom.md) for details.

## Converting
To generate multiple reports, you can generate the JSON report first and convert it to other formats with the `convert` subcommand.

```shell
$ trivy image --format json -o result.json --list-all-pkgs debian:11
$ trivy convert --format cyclonedx --output result.cdx result.json
```

!!! note
    Please note that if you want to convert to a format that requires a list of packages, 
    such as SBOM, you need to add the `--list-all-pkgs` flag when outputting in JSON.

[Filtering options](./filtering.md) such as `--severity` are also available with `convert`.

```shell
# Output all severities in JSON
$ trivy image --format json -o result.json --list-all-pkgs debian:11

# Output only critical issues in table format
$ trivy convert --format table --severity CRITICAL result.json
```

!!! note
    JSON reports from "trivy aws" and "trivy k8s" are not yet supported.

[cargo-auditable]: https://github.com/rust-secure-code/cargo-auditable/
[action]: https://github.com/aquasecurity/trivy-action
[asff]: ../../tutorials/integrations/aws-security-hub.md
[sarif]: https://docs.github.com/en/github/finding-security-vulnerabilities-and-errors-in-your-code/managing-results-from-code-scanning
[sprig]: http://masterminds.github.io/sprig/

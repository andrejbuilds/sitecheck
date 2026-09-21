<div align="center">

# Sitecheck

### A pre-deployment checker for websites and WordPress projects

Sitecheck scans a project folder and reports common risks before a website reaches production.

</div>

---

## About the project

Sitecheck is a Python command-line tool that scans website project folders for common deployment risks.

Website deployments often depend on memory and manual checklists. Sitecheck makes part of that process repeatable by checking for issues such as exposed environment files, enabled WordPress debugging, risky project artifacts, missing files, and suspicious PHP indicators.

Results are classified as `PASS`, `WARN`, or `FAIL` and can be displayed as readable terminal output or structured JSON.

Sitecheck reports potential risks and indicators—it does not claim to prove that a website has been compromised.

## Why I built it

The idea came from practical WordPress maintenance and deployment work.

Before publishing or migrating a website, developers often need to verify the same configuration, security, and project-hygiene details. Sitecheck explores how those repeated checks can be organized into a small, reusable command-line tool.

The project also gives me a practical way to learn foundational Python through a problem connected directly to my professional web development experience.

## Features

■ Generic website and WordPress-specific checks <br>
■ Automatic WordPress profile detection <br>
■ Human-readable terminal output <br>
■ Structured JSON output for automation <br>
■ `PASS`, `WARN`, and `FAIL` severity levels <br>
■ Overall deployment verdicts <br>
■ Filtered output by result status <br>
■ Compact summary output <br>
■ Optional deeper WordPress scanning <br>
■ Configurable ignored checks <br>
■ Meaningful process exit codes <br>
■ Automated test coverage with Pytest <br> 
■ GitHub Actions testing on pushes and pull requests <br>

## What Sitecheck reviews

### Generic website checks

| Area                 | What it reviews                                                              |
| -------------------- | ---------------------------------------------------------------------------- |
| **Path validation**  | Confirms that the supplied path exists and is a directory                    |
| **Git hygiene**      | Checks for a Git repository, `.gitignore`, and exposed `.env` files          |
| **Risky root files** | Reviews backup files, archives, database files, and public development files |
| **Redirects**        | Reviews `.htaccess` for external redirects                                   |
| **Dependencies**     | Checks Composer and npm package/lockfile consistency                         |
| **Local artifacts**  | Detects `node_modules`, editor directories, and operating-system files       |
| **Debug files**      | Reviews temporary files, debug artifacts, and error logs                     |

### WordPress checks

| Area                    | What it reviews                                                                                  |
| ----------------------- | ------------------------------------------------------------------------------------------------ |
| **WordPress structure** | Checks `wp-config.php`, `wp-content`, and partial WordPress installations                        |
| **Public files**        | Reviews `readme.html`, `xmlrpc.php`, `license.txt`, sample configuration, and installation files |
| **Debug settings**      | Checks `WP_DEBUG`, `WP_DEBUG_LOG`, `WP_DEBUG_DISPLAY`, `SCRIPT_DEBUG`, and `display_errors`      |
| **Hardening**           | Reviews `DISALLOW_FILE_EDIT` and `WP_ENVIRONMENT_TYPE`                                           |
| **Debug artifacts**     | Detects `wp-content/debug.log`                                                                   |
| **PHP indicators**      | Reviews PHP files in uploads and disguised PHP files inside plugins                              |
| **Suspicious patterns** | Reports potentially suspicious PHP patterns found in uploads                                     |
| **Deep scanning**       | Optionally checks unexpected PHP files in `wp-content` and its cache directory                   |

## Severity levels

| Status | Meaning                                                           |
| ------ | ----------------------------------------------------------------- |
| `PASS` | The check found no issue                                          |
| `WARN` | A possible risk was found and should be reviewed                  |
| `FAIL` | A blocking issue prevents the project from being considered ready |

Sitecheck also produces one overall verdict:

| Verdict               | Meaning                                              |
| --------------------- | ---------------------------------------------------- |
| `ready`               | No warnings or failures were found                   |
| `ready_with_warnings` | No failures were found, but some results need review |
| `not_ready`           | One or more blocking failures were found             |

Warnings are deliberately conservative. A suspicious filename or code pattern can have a legitimate explanation and should be reviewed manually.

## Requirements

* Python 3.11 or newer
* pip

Sitecheck has no external runtime dependencies.

## Installation

Clone the repository:

```bash
git clone https://github.com/andrejbuilds/sitecheck.git
```

Open the project directory:

```bash
cd sitecheck
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it on Windows:

```powershell
.venv\Scripts\activate
```

Activate it on macOS or Linux:

```bash
source .venv/bin/activate
```

Install Sitecheck with its development dependencies:

```bash
python -m pip install -e ".[dev]"
```

Confirm that it is installed:

```bash
sitecheck --version
```

## Usage

Display the available commands:

```bash
sitecheck --help
```

Scan the current directory:

```bash
sitecheck scan .
```

Scan another project:

```bash
sitecheck scan ./path-to-project
```

Run the optional deeper WordPress checks:

```bash
sitecheck scan . --deep
```

Show only warnings:

```bash
sitecheck scan . --only warn
```

Show only failures:

```bash
sitecheck scan . --only fail
```

Display a compact summary:

```bash
sitecheck scan . --summary
```

Return the complete scan as JSON:

```bash
sitecheck scan . --json
```

## Example output

```text
Detected profile: wordpress
Verdict: ready_with_warnings

WARN: PHP files found inside wp-content/uploads; review them before production deployment
  Details:
    - wp-content/uploads/2026/05/logo.png.php
    - wp-content/uploads/2026/05/shell.php

Summary:
PASS: 27
WARN: 6
FAIL: 0

Review WARN items before deployment.
```

## Output options

### Filter by status

Use `--only` to display individual results with a particular status:

```bash
sitecheck scan . --only pass
sitecheck scan . --only warn
sitecheck scan . --only fail
```

Filtering changes only what is displayed. The complete scan still runs, and the summary, verdict, and exit code remain unchanged.

### Summary mode

Use `--summary` for a shorter terminal view:

```bash
sitecheck scan . --summary
```

Example:

```text
Detected profile: generic
Verdict: ready

Summary:
PASS: 16
WARN: 0
FAIL: 0
```

### JSON output

Use `--json` to receive structured scan data:

```bash
sitecheck scan . --json
```

Example structure:

```json
{
  "path": ".",
  "profile": "wordpress",
  "results": [],
  "summary": {
    "pass": 0,
    "warn": 0,
    "fail": 0
  },
  "verdict": "ready"
}
```

JSON mode always returns the complete scan data. Display options such as `--only` and `--summary` do not remove results from the JSON output.

## Configuration

Create a `.sitecheck.toml` file in the root of the project being scanned to ignore checks that are not relevant to that project:

```toml
[ignore]
checks = ["xmlrpc", "node_modules"]
```

Ignored checks are removed from the results and summary counts.

Checks should be ignored intentionally. Hiding a warning does not resolve the underlying risk.

## Exit codes

Sitecheck returns process exit codes that can be used by scripts and automated workflows:

| Code | Meaning                                                                 |
| ---- | ----------------------------------------------------------------------- |
| `0`  | The scan completed without any `FAIL` results                           |
| `1`  | The command was invalid or the scan produced at least one `FAIL` result |

Warnings do not produce a failing exit code.

## Testing

The project currently contains **152 passing tests** covering:

* CLI commands and options
* Scanner behavior
* Profile detection
* Generic website checks
* WordPress-specific checks
* Summary and verdict generation
* Configuration and ignored checks
* Text and JSON output
* Exit-code behavior

Run the complete test suite:

```bash
python -m pytest
```

On Windows or OneDrive environments where temporary Pytest directories become locked, use a new temporary directory:

```powershell
$stamp = Get-Date -Format 'yyyyMMddHHmmssfff'
python -m pytest --basetemp ".pytest_tmp_$stamp"
```

## Continuous integration

A basic GitHub Actions workflow installs the package and runs the Pytest suite on:

* Pushes
* Pull requests

A failed test causes the workflow to fail, helping prevent broken changes from being merged unnoticed.

## Development approach

Sitecheck is an AI-assisted learning project combining my practical WordPress experience with foundational Python development.

I defined the problem, project requirements, checks, expected behavior, and practical deployment risks based on real website work. AI coding tools assisted with implementation, debugging, testing, review, and documentation.

I review the generated changes, run the test suite, verify behavior through practical scenarios, and refine the project incrementally.

## Project status

Sitecheck is an active, early-stage project that I continue developing during my free time.

The core CLI, generic checks, WordPress profile, output formats, configuration, test suite, and continuous-integration workflow are operational. The current focus is improving documentation, message clarity, and practical usefulness while keeping scans fast and avoiding unnecessary noise.

The project is currently pre-release and has not been published as a Python package.

## Roadmap

Planned improvements include:

* Improving existing warning messages and recommendations
* Adding more practical generic website checks
* Strengthening WordPress hardening checks
* Improving project documentation and examples
* Adding regression tests for new behavior
* Refining configuration without making it unnecessarily complex
* Preparing the project for a possible future release

Larger features will only be added when they solve a clear practical problem.

## License

Sitecheck is available under the [MIT License](LICENSE).

---

<div align="center">

Built from real WordPress deployment experience, one practical check at a time.

</div>

## Introduction


Snyk's Static Application Security Testing (SAST) tools analyze source code for vulnerabilities like injection flaws, XSS,
and more. Integrated into development workflows, it provides prioritized recommendations for remediation, enhancing code 
quality and security. With comprehensive reporting, teams gain visibility to track security initiatives and mitigate risks 
effectively, ensuring the delivery of secure software.

Supported languages:
- JavaScript/Node.js: Snyk offers extensive support for JavaScript and Node.js applications, including npm packages and dependencies.
- Java: Snyk provides support for scanning Java applications and libraries, including Maven and Gradle dependencies.
- Python: Snyk can analyze Python applications and dependencies, including packages managed via pip.
- Ruby: Snyk supports scanning Ruby applications and gems, ensuring the security of Ruby on Rails projects and other Ruby-based applications.
- Go: Snyk offers support for Go applications and modules, enabling developers to identify and remediate vulnerabilities in their Go codebases.
- PHP: Snyk can scan PHP applications and libraries, including Composer dependencies commonly used in PHP projects.
- .NET: Snyk provides support for .NET applications, including those built using C# and utilizing NuGet packages.
- Container Images: Snyk supports scanning container images for vulnerabilities, regardless of the programming language used within the containers.
- Infrastructure as Code (IaC): Snyk also offers support for scanning Infrastructure as Code templates written in languages such as YAML, JSON, Terraform, and CloudFormation.

## Implementation of a PoC (Proof of Concept)

# Github Action

As Snyk requires a public/private repository to scan the code (open to a Internet connection), the PoC was implemented in a personal Github Repository. 
Repo reference for tests to scan diferentes levels of vunerabilities: https://github.com/dbelob/helloworld-web
This repository contains multiple folders with different levels of vulnerabilities in the code, based in different versions of JAVA/Maven dependencies.

Github Action code to scan the repository:
```yaml
name: Example workflow for Maven using Snyk
#description: 'Snyk test demo for Maven 3.9.3/JAVA 17,linked to US - https://atc.bmwgroup.net/jira/browse/CAWE-790'

on: push
jobs:
  security:
    permissions:
      contents: read # for actions/checkout to fetch code
      security-events: write # for github/codeql-action/upload-sarif to upload SARIF results
      actions: read # only required for a private repository by github/codeql-action/upload-sarif to get the Action run status
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master

      - name: Install Snyk CLI
        run: npm install -g snyk && npm update -g snyk && npm i -g snyk-recursive && npm update -g snyk-recursive && echo "Snyk CLI already installed"

      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/maven-3-jdk-17@master
        continue-on-error: true # To make sure that SARIF upload gets called
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --sarif-file-output=snyk.sarif
          command: code test && snyk-recursive

      - name: Upload result to GitHub Code Scanning
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: snyk.sarif

      - name: Print the Snyk SARIF file
        run: |
          cat snyk.sarif
```




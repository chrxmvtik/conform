<!-- markdownlint-disable MD041 -->

<p align="center">
  <h1 align="center">Conform</h1>
  <p align="center">Policy enforcement for your pipelines.</p>
  <p align="center">
    <a href="https://conventionalcommits.org"><img alt="Conventional Commits" src="https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg?style=flat-square"></a>
    <a href="https://godoc.org/github.com/siderolabs/conform"><img alt="GoDoc" src="http://img.shields.io/badge/godoc-reference-blue.svg?style=flat-square"></a>
    <a href="https://travis-ci.org/siderolabs/conform"><img alt="Travis" src="https://img.shields.io/travis/siderolabs/conform.svg?style=flat-square"></a>
    <a href="https://codecov.io/gh/siderolabs/conform"><img alt="Codecov" src="https://img.shields.io/codecov/c/github/siderolabs/conform.svg?style=flat-square"></a>
    <a href="https://goreportcard.com/report/github.com/siderolabs/conform"><img alt="Go Report Card" src="https://goreportcard.com/badge/github.com/siderolabs/conform?style=flat-square"></a>
    <a href="https://github.com/siderolabs/conform/releases/latest"><img alt="Release" src="https://img.shields.io/github/release/siderolabs/conform.svg?style=flat-square"></a>
    <a href="https://github.com/siderolabs/conform/releases/latest"><img alt="GitHub (pre-)release" src="https://img.shields.io/github/release/siderolabs/conform/all.svg?style=flat-square"></a>
  </p>
</p>

---

**Conform** is a tool for enforcing policies on your build pipelines.

Some of the policies included are:

- **Commits**: Enforce commit policies including:
  - Commit message header length
  - Developer Certificate of Origin
  - GPG signature
  - GPG signature identity check
  - [Conventional Commits](https://www.conventionalcommits.org)
  - Imperative mood
  - Spell check
  - Maximum of one commit ahead of `master`
  - Require a commit body
  - Jira issue check
- **License Headers**: Enforce license headers on source code files.

## Getting Started

To install conform you can download a [release](https://github.com/siderolabs/conform/releases), or build it locally (go must be installed):

```bash
go install github.com/siderolabs/conform/cmd/conform@latest
```

Third option is to run it as a container:

```bash
docker run --rm -it -v $PWD:/src:ro,Z -w /src ghcr.io/siderolabs/conform:v0.1.0-alpha.22 enforce
```

You can also install conform with [aqua](https://aquaproj.github.io/).

```bash
aqua g -i siderolabs/conform
```

Now, create a file named `.conform.yaml` with the following contents:

```yaml
policies:
  - type: commit
    spec:
      header:
        length: 89
        imperative: true
        case: lower
        invalidLastCharacters: .
        jira:
          keys:
          - PROJ
          - JIRA
      body:
        required: true
      dco: true
      gpg:
        required: false
        identity:
          gitHubOrganization: some-organization
      spellcheck:
        locale: US
      maximumOfOneCommit: true
      conventional:
        types:
          - "type"
        scopes:
          - "scope"
        descriptionLength: 72
  - type: license
    spec:
      skipPaths:
        - .git/
        - .build*/
      includeSuffixes:
        - .ext
      excludeSuffixes:
        - .exclude-ext-prefix.ext
      allowPrecedingComments: false
      header: |
        This is the contents of a license header.
```

In the same directory, run:

```bash
$ conform enforce
POLICY         CHECK                        STATUS        MESSAGE
commit         Header Length                PASS          Header is 43 characters
commit         Imperative Mood              PASS          Commit begins with imperative verb
commit         Header Case                  PASS          Header case is valid
commit         Header Last Character        PASS          Header last character is valid
commit         DCO                          PASS          Developer Certificate of Origin was found
commit         GPG                          PASS          GPG signature found
commit         GPG Identity                 PASS          Signed by "Someone <someone@example.com>"
commit         Conventional Commit          PASS          Commit message is a valid conventional commit
commit         Spellcheck                   PASS          Commit contains 0 misspellings
commit         Number of Commits            PASS          HEAD is 0 commit(s) ahead of refs/heads/master
commit         Commit Body                  PASS          Commit body is valid
license        File Header                  PASS          All files have a valid license header
```

To setup a `commit-msg` hook:

```bash
cat <<EOF | tee .git/hooks/commit-msg
#!/bin/sh

conform enforce --commit-msg-file \$1
EOF
chmod +x .git/hooks/commit-msg
```

We also provide a [Pre-Commit](https://pre-commit.com) hook that you can use as follows:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/siderolabs/conform
    rev: master
    hooks:
      - id: conform
        stages:
          - commit-msg
```

### License

[![license](https://img.shields.io/github/license/siderolabs/conform.svg?style=flat-square)](https://github.com/siderolabs/conform/blob/master/LICENSE)

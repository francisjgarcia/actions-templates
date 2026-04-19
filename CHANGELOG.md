# Changelog

## [v1.0.5](https://github.com/francisjgarcia/actions-templates/releases/tag/v1.0.5) (2026-04-19)

### 🤖 AI-generated summary
In release v1.0.5, the handling of environment variables has been simplified, addressing a prior complexity that users faced. This change aims to improve usability and clarity, making it easier for users to manage environment settings within the application.


### 🐛 Bug fixes

* fix: simplify handling env (#23) ([c435e68](https://github.com/francisjgarcia/actions-templates/commit/c435e68)) — Francis J. García



## [v1.0.4](https://github.com/francisjgarcia/actions-templates/releases/tag/v1.0.4) (2026-04-18)

### 🤖 AI-generated summary
In release v1.0.4, the handling of sensitive environment variables within workflows has been simplified, making it easier for users to manage these variables securely. This improvement enhances the overall usability and security of the software, ensuring that users can focus on their tasks without worrying about complex configurations.
### 🐛 Bug fixes

* fix(workflow): simplify handling of sensitive environment variables (#22) ([9153a3f](https://github.com/francisjgarcia/actions-templates/commit/9153a3f)) — Francis J. García

## [v1.0.3](https://github.com/francisjgarcia/actions-templates/releases/tag/v1.0.3) (2026-04-18)

### 🤖 AI-generated summary
In release v1.0.3, a bug fix was implemented to update a secret variable, addressing an issue that may have affected user experience. This change improves the security and functionality of the application, ensuring that sensitive information is handled correctly.
### 🐛 Bug fixes

* fix: update secret variable (#21) ([f7d9381](https://github.com/francisjgarcia/actions-templates/commit/f7d9381)) — Francis J. García

## [v1.0.2](https://github.com/francisjgarcia/actions-templates/releases/tag/v1.0.2) (2026-04-13)

### 🤖 AI-generated summary
In release v1.0.2, a key bug fix was implemented to ensure that CodeQL scans are triggered correctly when changes are pushed to the main branch. This improvement enhances the workflow's reliability, helping users maintain better code quality and security by ensuring vulnerabilities are detected promptly.
### 🐛 Bug fixes

* fix(workflow): ensure CodeQL runs on push to main branch (#20) ([ab7466a](https://github.com/francisjgarcia/actions-templates/commit/ab7466a)) — Francis J. García

## [v1.0.1](https://github.com/francisjgarcia/actions-templates/releases/tag/v1.0.1) (2026-04-13)

### 🤖 AI-generated summary
In release v1.0.1, the main update involves a bug fix where the version of the GitHub Actions dependency "softprops/action-gh-release" was updated. This improvement is significant as it enhances the reliability and performance of the associated workflows, ensuring smoother automation for users.
### 🐛 Bug fixes

* fix(deps): bump softprops/action-gh-release in the github-actions group (#19) ([475c0c7](https://github.com/francisjgarcia/actions-templates/commit/475c0c7)) — dependabot[bot]

## [v1.0.0](https://github.com/francisjgarcia/actions-templates/releases/tag/v1.0.0) (2026-04-11)

### 🤖 AI-generated summary
In the v1.0.0 release, a significant breaking change was introduced with an update to the model, which needs users to adapt their implementations accordingly. This change aims to improve overall functionality and performance, ensuring a better user experience moving forward. Users should review the details to understand how the adjustments may impact their current setups.
### ⚠️ Breaking changes

* feat!: Feat/fix model2 (#18) ([f59675c](https://github.com/francisjgarcia/actions-templates/commit/f59675c)) — Francis J. García

## [v0.10.0](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.10.0) (2026-04-11)

### 🤖 AI-generated summary
In release v0.10.0, the AI-generated summary feature in release notes has been improved, enhancing its effectiveness and clarity for users. This change aims to provide better insights into updates, making it easier for users to understand the key changes and benefits of each release.
### ✨ New features

* feat(workflows): enhance AI-generated summary in release notes (#17) ([3f6f1da](https://github.com/francisjgarcia/actions-templates/commit/3f6f1da)) — Francis J. García

## [v0.9.0](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.9.0) (2026-04-11)

In the v0.9.0 release, a new feature was introduced for "Prueba," enhancing the overall functionality of the software. Additionally, a critical bug related to permissions in the continuous integration (CI) release job was fixed, ensuring that models have the necessary read access, which improves the reliability of the deployment process. These updates aim to streamline user experience and enhance the software's performance.
### ✨ New features

* feat(fix): Prueba (#15) ([39bb4f4](https://github.com/francisjgarcia/actions-templates/commit/39bb4f4)) — Francis J. García

### 🐛 Bug fixes

* fix(ci): add read permission for models in release job (#16) ([d8addcb](https://github.com/francisjgarcia/actions-templates/commit/d8addcb)) — Francis J. García

## [v0.8.3](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.8.3) (2026-04-11)
### 🐛 Bug fixes

* fix(test): test (#14) ([41cd5c4](https://github.com/francisjgarcia/actions-templates/commit/41cd5c4)) — Francis J. García

## [v0.8.2](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.8.2) (2026-04-11)
### 🐛 Bug fixes

* fix: remove unnecessary lines from PR template (#13) ([d063916](https://github.com/francisjgarcia/actions-templates/commit/d063916)) — Francis J. García

## [v0.8.1](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.8.1) (2026-04-11)
### 🐛 Bug fixes

* fix(ia): test (#12) ([ba8789e](https://github.com/francisjgarcia/actions-templates/commit/ba8789e)) — Francis J. García

## [v0.8.0](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.8.0) (2026-04-11)
### ✨ New features

* feat(workflows): Ia (#11) ([b55a00f](https://github.com/francisjgarcia/actions-templates/commit/b55a00f)) — Francis J. García

## [v0.7.0](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.7.0) (2026-04-11)
### ✨ New features

* feat(workflows): add target_commitish to GitHub release API call (#10) ([218d070](https://github.com/francisjgarcia/actions-templates/commit/218d070)) — Francis J. García

## [v0.6.0](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.6.0) (2026-04-11)
### ✨ New features

* feat: Test (#9) ([8eca4fd](https://github.com/francisjgarcia/actions-templates/commit/8eca4fd)) — Francis J. García
* feat(workflows): enhance container configuration description and manage sensitive env vars (#8) ([f0b2daf](https://github.com/francisjgarcia/actions-templates/commit/f0b2daf)) — Francis J. García
* feat(lint): add YAML front matter to lint workflow (#4) ([d654cd4](https://github.com/francisjgarcia/actions-templates/commit/d654cd4)) — Francis J. García

### 🔧 Other changes

* ci(tests): tests (#7) ([1d91c2e](https://github.com/francisjgarcia/actions-templates/commit/1d91c2e)) — Francis J. García
* hola(workflows): update action versions to latest stable releases (#6) ([ea1e567](https://github.com/francisjgarcia/actions-templates/commit/ea1e567)) — Francis J. García
* Update CodeQL workflow configuration (#5) ([149cfcf](https://github.com/francisjgarcia/actions-templates/commit/149cfcf)) — Francis J. García

## [v0.5.4](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.5.4) (2026-04-04)
### 🐛 Bug fixes

* fix(README): update language support in key features section ([154c6a4](https://github.com/francisjgarcia/actions-templates/commit/154c6a4)) — Francis J. García

### 🔧 Other changes

* Merge pull request #3 from francisjgarcia/test ([741534c](https://github.com/francisjgarcia/actions-templates/commit/741534c)) — Francis J. García

## [v0.5.3](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.5.3) (2026-04-04)
### 🐛 Bug fixes

* fix(workflows): update yamllint summary to use inline paths ([bb013b0](https://github.com/francisjgarcia/actions-templates/commit/bb013b0)) — Francis J. García
* fix(workflows): simplify lint summary output ([67db083](https://github.com/francisjgarcia/actions-templates/commit/67db083)) — Francis J. García
* fix(workflows): update lint summary to include paths ([28b3eed](https://github.com/francisjgarcia/actions-templates/commit/28b3eed)) — Francis J. García
* fix(workflows): correct API URL formatting for repo description retrieval ([6b53a92](https://github.com/francisjgarcia/actions-templates/commit/6b53a92)) — Francis J. García
* fix(workflows): simplify summary generation in multiple workflow files ([4287357](https://github.com/francisjgarcia/actions-templates/commit/4287357)) — Francis J. García
* fix(workflows): remove actionlint paths input and simplify summary ([a3c78f7](https://github.com/francisjgarcia/actions-templates/commit/a3c78f7)) — Francis J. García

### 🔧 Other changes

* Merge pull request #2 from francisjgarcia/test ([53f3434](https://github.com/francisjgarcia/actions-templates/commit/53f3434)) — Francis J. García

## [v0.5.2](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.5.2) (2026-04-04)
### 🐛 Bug fixes

* fix(docs): remove Node.js usage guide from README.md ([aadd424](https://github.com/francisjgarcia/actions-templates/commit/aadd424)) — Francis J. García

### 🔧 Other changes

* Merge pull request #1 from francisjgarcia/test ([723f214](https://github.com/francisjgarcia/actions-templates/commit/723f214)) — Francis J. García

## [v0.5.1](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.5.1) (2026-04-04)
### 🐛 Bug fixes

* fix(workflows): remove unnecessary yamllint rule ([50c50ed](https://github.com/francisjgarcia/actions-templates/commit/50c50ed)) — Francis J. García

## [v0.5.0](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.5.0) (2026-04-04)
### ✨ New features

* feat(workflows): update linting configuration for YAML files ([762bfe7](https://github.com/francisjgarcia/actions-templates/commit/762bfe7)) — Francis J. García

## [v0.4.0](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.4.0) (2026-04-04)
### ✨ New features

* feat(workflows): add YAML front matter to workflow files ([3203603](https://github.com/francisjgarcia/actions-templates/commit/3203603)) — Francis J. García

## [v0.3.0](https://github.com/francisjgarcia/actions-templates/releases/tag/v0.3.0) (2026-04-04)
### ✨ New features

* feat(workflows): update `wf-release` to enhance CHANGELOG.md handling ([c4aaf34](https://github.com/francisjgarcia/actions-templates/commit/c4aaf34)) — Francis J. García

---

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<!-- releases will be prepended here automatically by wf-release.yml -->

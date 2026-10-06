# PR verification in TeamCity Pipelines

This pilot runs the same Gradle command and Java distribution as
`.github/workflows/pr-verification.yml` in the `dmitry-treskunov/ideavim` fork.
Property-based and long-running tests remain excluded.

[Open the pipeline on dima.teamcity.com](https://dima.teamcity.com/buildConfiguration/Sandbox_IdeaVimPilot_PrVerification?mode=builds).

The pipeline definition is in `pr-verification.yml`. It uses a Linux-Medium
agent, Amazon Corretto JDK 21 and a shared Gradle dependency/wrapper cache.
JUnit XML is imported into TeamCity's Tests view. Only Gradle's
`build/reports/problems/problems-report.html` is published as a build artifact
when it exists, including when the test command fails. HTML test reports and
individual XML files are not published, avoiding thousands of duplicate report
files and the server's artifact-count limit.

The server-side GitHub integration discovers pull requests targeting `master`,
automatically triggers verification, and publishes the result back to GitHub.
Authentication uses the existing TeamCity GitHub connection; no credentials
are committed to this repository.

This pilot discovers PRs from repository members and verifies their head commit
(`refs/pull/<number>/head`). The GitHub Actions checkout uses GitHub's synthetic
merge commit, so compare these runs with an unchanged target branch.

The YAML is stored on the server for this pilot. After changing the file,
validate and apply it with the TeamCity CLI:

```bash
export TEAMCITY_URL=https://dima.teamcity.com
teamcity pipeline validate .teamcity/pr-verification.yml
teamcity pipeline push Sandbox_IdeaVimPilot_PrVerification .teamcity/pr-verification.yml
```

After pushing, check that the Pull Requests and Commit Status Publisher features
are enabled in the pipeline's build configuration. With TeamCity CLI 1.5.0 on
this server, a YAML push disabled both features; they had to be re-enabled with
their existing settings before PR discovery and GitHub status updates resumed.

PR discovery, the VCS trigger (PRs only, 30-second quiet period), and the GitHub
Commit Status Publisher are configured on the server separately from this YAML.
The trigger uses the logical branch filter `-:*` / `+:pull/*`; the Pull Requests
feature filters target branches with `+:refs/heads/master`.

The GitHub Actions workflow remains available for comparison during the pilot.

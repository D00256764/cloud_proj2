# copilot-instructions for cloud_proj2

Purpose: provide concise, actionable context so an AI coding agent can be productive immediately working on this repo.

Big picture
- This repo holds a small static website and CI/CD pipeline configuration: GitHub -> AWS CodePipeline -> CodeBuild -> S3 (website bucket).
- Artifacts bucket: `cloud-infra-proj2-artifacts`. Website bucket: `cloud-infra-proj2` (see `secure-pipeline.json` and `secure-codebuild-project.json`).
- Build is minimal: static HTML. `buildspec.yml` shows no compilation steps; CodeBuild packages files and stores an artefact ZIP.
- Index and error pages are in the repo root (`index.html`, `error.html`). CloudFront is referenced in the site text but there is no CloudFront template in this repo.

Key files (what to read first)
- `buildspec.yml` — CodeBuild steps (install/build/post_build). Useful to change build behavior or add tests.
- `secure-pipeline.json` — CodePipeline definition: Source (GitHub), Build (CodeBuild project `secure-website-build`), Deploy (S3). Contains GitHub repo/branch and an OAuthToken (sensitive).
- `secure-codebuild-project.json` — CodeBuild project settings (image, role, artifact S3 bucket, buildspec path).
- `pipeline-s3-policy.json` and `website-bucket-policy.json` — IAM / S3 policies applied to artifacts and website buckets (see permissions and public-read settings).
- `index.html`, `error.html` — the website content and the primary artifacts being deployed.

Developer workflows & quick commands
- Local preview: just open `index.html` in a browser. No local build required.
- To change pipeline behavior: edit `secure-pipeline.json` then update pipeline with AWS CLI / CloudFormation as your workflow requires. (This repo currently stores pipeline JSON; deployment is external.)
- To change CodeBuild steps: edit `buildspec.yml`. Example: add a test step under `build.commands`.

Important patterns & conventions
- Naming: buckets use `cloud-infra-proj2` and `cloud-infra-proj2-artifacts` consistently. CodeBuild project: `secure-website-build`.
- Artifacts: CodeBuild packages everything in repo per `artifacts.files: '**/*'` and uploads a ZIP to the artifacts bucket.
- Minimal pipeline: pipeline delegates compilation to CodeBuild but current buildspec assumes static content (no transpile/packaging steps). Add explicit commands if introducing JS/CSS builds.

Security notes (must not be ignored)
- `secure-pipeline.json` currently contains a GitHub OAuth token (`OAuthToken` field). Treat this as a secret: rotate immediately and move to a secure store (AWS Secrets Manager / SSM Parameter Store) referenced by the pipeline, or use CodePipeline GitHub integration with a connection.
- `website-bucket-policy.json` grants `s3:GetObject` to `Principal: *` for `arn:aws:s3:::cloud-infra-proj2/*` — this is expected for a public static website, but verify that CloudFront/HTTPS is enforced in your infra outside this repo.

Examples for common tasks
- Add a test to buildspec (append under `build.commands`):

  - run: npm ci && npm test

- Change deploy target bucket in pipeline: update `secure-pipeline.json` stage `Deploy` -> `configuration.BucketName` and ensure IAM roles/policies referenced (see `pipeline-s3-policy.json`) allow PutObject for that bucket.

When to ask the human
- If you need the canonical deployment method (CloudFormation / Terraform / CLI) — this repo contains JSON snippets but not a canonical deploy script. Ask which infra state is authoritative.
- Confirm whether GitHub token should be considered intentionally committed (likely accidental). If accidental, request rotation and removal.

Finish: After edits, open a PR against `main` and mention the pipeline/CodeBuild artifacts changes in the PR description so reviewers can verify IAM and S3 changes.

If anything above is unclear or you want additional examples (e.g., how to add CloudFront invalidation to the pipeline or how to switch to a GitHub Apps connection), tell me which area to expand.

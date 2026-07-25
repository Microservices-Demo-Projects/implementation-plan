# CATEGORY-01 — Simple Changes

Small POCs and single-repo refinements. Each item below is scoped to one repo
and a short checklist.

## 1.1 Serverless `expense-tracker`

Serverless expense-tracking POC with a full Infrastructure-as-Code setup and
automated backup / restore.

- [ ] Full Terraform setup (use Floci Emulator for local testing)
- [ ] Async Lambda automation for periodic S3 backup of DynamoDB data
- [ ] Async Lambda automation for restoring DynamoDB data from S3 backups
- [ ] CD pipeline for deployment automation

## 1.2 SFTP Server Implementation & Containerization

Two container images of the same SFTP server that show two different security
postures.

- [ ] **Variant A** — Multi-user SSH/SFTP with per-user chroot-jail directories;
      the SFTP daemon itself runs as `root`
- [ ] **Variant B** — Single shared non-root user for SSH/SFTP; filesystem is
      read-only everywhere except the target Kubernetes volume mount

## 1.3 Instrumentation Agent — Time Travel

A Java instrumentation agent that lets an application observe fake `Date` /
`Instant` / `Clock` values (past or future) without any application code
change.

- [ ] Byte-code time-travel implementation (mock `Date` / `Clock` at the agent
      level)
- [ ] Configurable offset / fixed instant via agent args or a control endpoint

## 1.4 AWS LightSail Demo Repo — Showcase

The AWS LightSail demo repository already exists; this task is about finishing
its documentation so it is presentable.

- [ ] Complete `README.md` and architectural notes
- [ ] Add screenshots / diagrams

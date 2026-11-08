---
name: gcp-safety
description: "Guard GCP and Google Workspace operations. Trigger: gcloud, gsutil, bq, kubectl on GKE, terraform on GCP, service account keys, Drive/Workspace API, or any Google Cloud console action."
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## When to Use

Load this skill before ANY interaction with Google Cloud or Google Workspace: `gcloud`, `gsutil`, `gcloud storage`, `bq`, `kubectl` against GKE, `terraform` with the Google provider, the Cloud Console, or any SDK call authenticated by a service account key.

Load it for reads too. The identity pre-flight below is cheap, and it is what tells you whether the next command is aimed at production.

This skill extends `destructive-action-guard`. That one is the law; this one is the jurisprudence.

## Identity Pre-Flight

Run before the first command of every session. You cannot reason about blast radius without knowing who you are and where you are pointed.

```bash
gcloud auth list                    # which identities exist, which is active
gcloud config get-value account     # who am I
gcloud config get-value project     # where am I pointed
```

When a service account key file is involved, resolve its identity WITHOUT printing the private key:

```bash
# Identity fields only. Never cat a credential file.
rg -o '"(client_email|client_id|project_id)": "[^"]*"' path/to/credentials.json
```

`client_id` in that file is the IAM `uniqueId`. It is the same value the Drive Activity API reports as `people/<id>`, and it is how a service account gets attributed in audit logs. Map it back with:

```bash
gcloud iam service-accounts list --project <PROJECT_ID>
```

**If the active project or the key's `project_id` is a production project, no destructive verb runs without explicit user confirmation. No exceptions.**

## Destructive Verbs, Reversible Alternative, Real Recovery Window

| Service | Destructive | Use instead | Recovery window |
|---|---|---|---|
| **Drive / shared drive** | `files.delete` | `files.update(trashed=true)` | Hard delete: admin-only restore, ~25 days, by a Workspace admin of the tenant. Trash: 30 days, user-recoverable |
| **Cloud Storage** | `gcloud storage rm -r`, `gsutil rm -r` | object versioning, or a soft-delete policy | Soft delete only if a policy is set (7 days default on newer buckets). Without it, none |
| **BigQuery** | `bq rm -r -f <dataset>` | drop tables individually, snapshot first | Table time travel, 7 days default, configurable 2-7. Dataset-level recovery is limited, do not rely on it |
| **Compute Engine** | `instances delete` | `instances stop` | None. Attached disks with auto-delete go too. `--keep-disks` saves the data, not the VM |
| **Persistent Disk** | `disks delete` | snapshot, then detach | None without a prior snapshot |
| **Cloud SQL** | `sql instances delete` | `--deletion-protection`, export first | Instance backups are deleted WITH the instance. Only exports survive |
| **GKE** | `container clusters delete` | resize node pool to 0 | None |
| **Project** | `projects delete` | apply a lien, or just stop billing | 30-day pending state, `projects undelete` restores. After that, gone |
| **IAM** | `remove-iam-policy-binding`, `service-accounts delete` | preview first where supported | None. Save the policy with `get-iam-policy` FIRST |
| **Secret Manager** | `secrets delete`, `versions destroy` | `versions disable` | None |
| **Artifact Registry** | `artifacts ... delete` | tag retention policies | None |
| **Pub/Sub** | `topics delete` | pause the subscription | None. Subscriptions orphan, in-flight messages are lost |
| **Terraform** | `destroy`, any plan with `replace` | `prevent_destroy` lifecycle block | Whatever the underlying service offers, which the plan output does not tell you |

Retention numbers change. **Verify against current provider docs before relying on any of them as a recovery plan.** And never read "there is a window" as permission to be careless: a window you cannot execute yourself is not a safety net.

## Google Workspace and Drive

Drive traps to avoid:

- **A shared drive id is also the id of its root folder.** Creating an item with no explicit parent puts it at the root of a real drive, as a sibling of production data.
- **`files.delete` on a shared drive is permanent.** It does not pass through the trash. No user-visible undo, and no trash entry to find later.
- **A permanent folder delete takes every descendant with it,** and the Activity API logs only the parent, so nobody can reconstruct what was lost.
- **Drive logs the service account, never the human who invoked it.** Attribution of the person lives in Cloud Audit Logs of the owning project, or nowhere.

| Rule | Requirement |
|---|---|
| Explicit parent, always | Never let `parent_id` fall through to a drive root. Set an explicit sandbox folder id for anything experimental. |
| Sandbox drive for fixtures | Test against a throwaway shared drive. Never one that holds real work. |
| `trashed=true` for teardown | Reserve `files.delete` for items you can prove this run created, and prefer the trash even then. |
| Least privilege for the SA | An integration account rarely needs Organizer or Manager. Content manager or Viewer covers most jobs. Organizer is what makes permanent deletion possible. |
| Config is not a boundary | A default parent in `appsettings` limits where things land by convention, not by permission. The real boundary is what the credential can reach. |

## Never Without Asking

Stop and ask, every time, regardless of prior approval in the session:

- Anything from the Destructive Verbs table against a non-sandbox project or drive.
- `gcloud projects delete`, organization writes, org policy changes.
- IAM removals, service account deletion, key deletion.
- Any recursive delete: `rm -r`, `bq rm -r`, `--recursive`, `--force`, `-f`.
- `terraform apply` where the plan reports resources to destroy or replace.
- Any command whose target you resolved from a wildcard, a list call, or a variable you did not set in this session.
- Using a production service account key for anything experimental.

## Safer Command Patterns

```bash
# Always print where you are before acting
gcloud config get-value project && gcloud config get-value account

# Pin the project per command instead of trusting ambient config
gcloud <group> <verb> --project <EXPLICIT_PROJECT_ID>

# Preview before mutating
terraform plan -out=tf.plan && terraform show tf.plan   # read it, then apply THAT file

# Save IAM policy before touching bindings
gcloud projects get-iam-policy <PROJECT_ID> --format=json > iam-backup.json

# Turn on the guardrails the platform gives you for free
gcloud sql instances patch <INSTANCE> --deletion-protection
gcloud storage buckets update gs://<BUCKET> --versioning
```

## Related

- `destructive-action-guard` — the general rules, pre-flight checklist, and confirmation gate.

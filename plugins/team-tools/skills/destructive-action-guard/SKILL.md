---
name: destructive-action-guard
description: "Stop before any irreversible action. Trigger: delete, drop, purge, truncate, rm, force push, destroy, terminate, revoke, overwrite, cleanup, teardown, reset — in any CLI, API, SDK, script, or console."
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## When to Use

Load this skill BEFORE executing anything that removes, overwrites, or reduces access to data or infrastructure. Any tool, any provider, any language, any environment.

Triggers include:

- **Filesystem** — `rm`, `rm -rf`, `Remove-Item -Recurse`, `del /s`, `mv` over an existing path, `>` redirect onto a real file.
- **Version control** — `git push --force`, `git reset --hard`, `git clean -fd`, branch or tag deletion, history rewrites, `git checkout .`
- **Databases** — `DROP`, `TRUNCATE`, `DELETE` without a bounded `WHERE`, destructive migrations, restoring a dump over a live database.
- **Cloud and infra** — deleting buckets, objects, instances, clusters, datasets, projects, DNS records, IAM bindings. `terraform apply` on any plan containing `destroy` or `replace`.
- **Containers** — `docker system prune`, volume removal, `kubectl delete`.
- **SaaS and document APIs** — Drive, SharePoint, S3, Notion, ticket systems: any delete, purge, or archive call.
- **Any routine named** `cleanup`, `teardown`, `reset`, `wipe`, `purge`, `reseed`, `afterAll`, `finally`.

That last category matters most. **Most production data loss does not come from someone deliberately typing a destructive command. It comes from cleanup code doing exactly what it was written to do, against the wrong target.**

## The Hard Rules

| # | Rule |
|---|------|
| 1 | **Reversible primitive first.** Trash over purge. Disable over delete. Stop over terminate. Rename over overwrite. Soft-delete over hard-delete. Revoke over destroy. Reach for the irreversible verb only when the reversible one has been explicitly ruled out AND the user approved that specific call. |
| 2 | **Never enumerate-then-delete.** Do not list a container and delete what comes back. Enumeration cannot distinguish your artifacts from someone else's. Delete only identifiers this run created and recorded. |
| 3 | **Fixtures never share a namespace with production.** Not the same bucket, drive, database, resource group, or folder. A separate prefix is not separation. |
| 4 | **State the blast radius before executing** — exact identifiers and exact count, not a category name. "the test folders" is not an inventory. |
| 5 | **Confirm before any irreversible action** on anything not provably a disposable sandbox. |
| 6 | **The recovery path is part of the plan.** Before executing, know: is there an undo, how long is the window, and who is authorized to run it. If the answer is "an admin we do not control", treat the action as permanent. |
| 7 | **Production credentials are not test credentials.** Holding a key is not authorization to experiment with it. |
| 8 | **Dry run first** whenever the tool offers one. |
| 9 | **Scope by identity, not by intention.** The question is not "what do I mean to touch" — it is "what can this credential reach if my filter is wrong". |
| 10 | **An uncommitted script is still a production action.** Code review and git history do not protect what was never committed. Ad-hoc scripts deserve MORE scrutiny, not less. |

## Pre-Flight Checklist

Answer all seven, explicitly, before the destructive call. Out loud, in the response — not silently.

1. **WHAT** — the exact identifiers and the exact count.
2. **WHERE** — which project, drive, database, cluster, branch, namespace. Print it from the tool; do not assume it from memory or config.
3. **WHO** — which credential or account executes this. Print it.
4. **REVERSIBLE?** — undo path, retention window, who can execute the recovery.
5. **DID THIS RUN CREATE IT?** — if no, stop and ask. Always.
6. **IS A REVERSIBLE PRIMITIVE AVAILABLE?** — if yes, use it instead.
7. **IF MY FILTER IS WRONG, WHAT ELSE MATCHES?** — name the neighbours.

**Any answer of "I am not sure" is a stop.** Not sure is a no. Ask the user one question and wait.

## Teardown: the correct shape

```text
# WRONG — enumerate and delete. Cannot tell your fixtures from real data.
for item in list_children(container):
    delete(item.id)

# WRONG — filter by name. Names collide, and real data can match your pattern.
for item in list_children(container):
    if item.name.startswith("test-"):
        delete(item.id)

# RIGHT — track what you create, undo in reverse, reversible primitive.
created = []
try:
    created.append(create(...))     # record EVERY id at creation time
    ...
finally:
    for id in reversed(created):
        trash(id)                   # not delete(); leave a recovery window
```

Three properties make the right version safe: the id list is **closed** (only what this run made), the primitive is **reversible**, and it is **order-aware** (children before parents).

## Confirmation Gate

When rule 5 fires, print this and stop. Do not proceed on silence, and do not proceed on a general prior approval given for a different action.

```text
IRREVERSIBLE ACTION — CONFIRMATION REQUIRED

Action:      <exact command or API call>
Identity:    <account / credential executing>
Target:      <project / drive / database / cluster>
Affects:     <N> item(s):
             - <id> <name>
             - <id> <name>
Reversible:  <no | yes, N days, recoverable by WHOM>
If wrong:    <what else is in blast radius>

Reply "confirmo" to proceed.
```

## Anti-Patterns

| Anti-pattern | Why it burns |
|---|---|
| "It is just test data" | The classifier for "test" is your assumption, not a property of the data. |
| "The name has a test prefix" | Real data can match. Names are not identity. |
| "I will clean it up after" | Cleanup is exactly where the accident happens. Design it before you create anything. |
| "I have the credential, so I am allowed" | Access is not authorization. Least privilege applies to you too. |
| "Delete is fine, there is a trash" | Verify it. Many APIs offer a hard-delete that bypasses trash entirely, on the same method name. |
| "I will point it at production just this once" | There is no "once" for an irreversible call. |
| Approval reuse | Approval for one destructive action never carries to the next one. |

## Related

- `gcp-safety` — GCP, gcloud, gsutil, bq, and Google Workspace API specifics.

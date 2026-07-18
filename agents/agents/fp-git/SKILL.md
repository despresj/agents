---
name: fp-git
description: Diagnose, recover, and safely rewrite Git history by reasoning from objects, refs, the index, and the working tree before destructive action. Use when work appears lost after rebase, reset, amend, or force-push; resolving conflicts; recovering commits; untangling repository state; rewriting shared history; or removing committed secrets or large files.
---

# First Principles — Git

Apply the `first-principles` core skill. Use this pack for Git invariants, traps,
and preconditions; use the core once for checkpoints and reporting.

## The Invariants

Reason from these, not from remembered commands:

1. **Commits are immutable snapshots in a content-addressed DAG.** Rebase and
   amend create new commits; reset and branch operations move refs. An old commit
   is recoverable only while some clone still has the object and it has not been
   pruned. Local reflogs commonly retain unreachable tips for a limited,
   configurable period—inspect them promptly and do not run cleanup first.
2. **Branches are just movable pointers to commits.** "Losing a branch" means
   losing a ref; whether the commits survive is a separate, inspectable question.
3. **History rewriting creates new commits; the old ones still exist** — locally
   until pruned in clones that fetched them. This is why removing a secret from
   visible history does not revoke or un-distribute it.
4. **HEAD, index, and working tree are distinct states.** `reset`, `restore`,
   `checkout`, and `commit` move or copy between different states. Inspect all
   three before choosing a command from memory.

## Trap Table

| Symptom | Tempting move | First-principles move | Discriminating test |
|---------|--------------|----------------------|---------------------|
| "My/teammate's commits are gone" after rebase/reset/force-push | Re-do the work | Search refs and object stores before assuming the commit survived or vanished | Inspect `git reflog --all`, `git log --all`, the affected clone's reflog, remotes, and other clones; verify a candidate SHA before creating a rescue branch |
| Merge/rebase conflict | Pick a side that compiles | A conflict is two *intents*, not two texts; resolve the intent | Read both commits (`git log -p`) and state what each was trying to do before editing |
| Repo state confusing / "git is broken" | Delete and re-clone | Inventory refs, HEAD, index, worktree, and operation state | Inspect `git status`, both staged and unstaged diffs, `git log --graph --oneline --all`, and reflogs before changing state |
| Need to update a shared rewritten branch | `push --force` | Protect the exact remote tip reviewed, not merely the last fetched tracking ref | Record the remote SHA and use `--force-with-lease=<ref>:<expected-sha>` after coordinating consumers |
| Secret committed in history | Rewrite history, done | Invariant 3: every fetch already has it | **Rotate the credential first**; then `git filter-repo`; then coordinate re-clones |
| "Which commit broke it?" | Read diffs and guess | Bisect only when a reliable test separates a known-good from a known-bad revision | Verify both endpoints and test determinism, then run `git bisect run <test>` |
| `.gitignore` added but file still tracked | Add more patterns | Ignore rules apply only to untracked files | `git rm --cached <file>`, then ignore |
| Detached HEAD warning | Make random checkouts | Treat it as a HEAD state, not damage; preserve any new commits before moving away | Inspect `git status` and `git log`; create a branch at the intended commit before switching |
| Rebase vs merge debate | Team habit / dogma | Derive from what history is *for* here: review, bisect, audit | Does the target reader need the true chronology or a clean narrative? |

## Precondition Template (O1)

**"This history rewrite is acceptably contained":** identify the exact refs and
commits to replace; record the current local and remote SHAs; coordinate known
consumers; create a local rescue ref unless retaining the old history would itself
preserve sensitive data; verify the rewritten graph and content before pushing;
protect the explicit expected remote SHA with `--force-with-lease=<ref>:<sha>`;
rotate leaked credentials before treating history cleanup as remediation.

## Domain Escalation Triggers

- About to run a destructive command (`reset --hard`, `clean -fd`, `push
  --force`, history filter) → resolve the exact target, inspect status and diffs,
  and state what becomes unreachable and where recovery lives before running it.
- About to re-clone, run garbage collection, or re-do work → stop and inspect
  reflogs, refs, unreachable objects, remotes, and other clones first.
- A secret is involved → the clock started at push time, not discovery time;
  rotation outranks history cleanup.

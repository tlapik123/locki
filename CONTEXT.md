# Locki

Sandboxed git worktrees for AI agents: each conversation gets a worktree and an
Incus container in a shared Lima VM, isolated from the host repo and home.

## Language

**Sandbox**:
A worktree plus its Incus container; the unit users create, enter, stop, and remove.
_Avoid_: container, VM, environment

**Worktree**:
The host-side git worktree of a sandbox, bind-mounted into its container at the
same absolute path. Can outlive its container (e.g. after `locki vm delete`).
_Avoid_: checkout, workspace

**Container**:
The Incus half of a sandbox, named by the sandbox id. Disposable — recreated on
demand from the worktree.

**Sandbox id**:
The 8-character slug shared by the worktree directory (`<repo>-locki-<id>`), the
branch suffix (`#locki-<id>`), and the container name.
_Avoid_: treating "worktree id" and "container name" as distinct identifiers —
they are all the same id

**Host repo**:
The user's original clone that worktrees are created from (`WorktreeInfo.repo`).
_Avoid_: source repo, main repo, master folder

**Include**:
A second repo's worktree grafted into a sandbox under `.locki/include/`.

**Trunk**:
The repo's main branch: origin/HEAD's target, falling back to local main/master.

**Bridge**:
The host daemon executing the filtered git/gh/locki command set on behalf of a
sandbox. Anything not in the filter runs host-side only.
_Avoid_: proxy

**Pull / Push**:
File transfer between a worktree and its host repo, 1:1 by relative path --
pull is worktree→host repo, push is host repo→worktree (`locki file`).
_Avoid_: sync, copy out/in

**Dirty**:
Carrying uncommitted changes: a dirty worktree makes `locki rm` offer a rescue
(and blocks non-interactive removal); `--dirty` creates a sandbox seeded with
the host repo's uncommitted state. `--raw` is `--dirty` plus gitignored files
(the raw repo folder).

**Rescue**:
The interactive offer in `locki rm` to pull files the removal would destroy.

**Clash**:
A transfer destination whose overwrite would lose content git cannot restore:
untracked with different content, tracked with uncommitted modifications, an
uncommitted deletion, or a directory/symlink standing in the file's place.
A file differing only in the executable bit differs (git tracks that bit).
_Avoid_: conflict (that's git merges), collision

**Idle**:
A container with no live Incus operation. Idle containers are *stopped*, never
deleted.

**Last used**:
When a sandbox was last opened (`ai`/`x`/`cd`/`ide`) or had container activity.
Not the worktree's file mtimes.
_Avoid_: last active (that is the daemon's transient bookkeeping for running
containers)

**Orphan**:
A container whose worktree no longer exists on disk; reaped automatically.

**Stop**:
Reversible shutdown of a sandbox's container; worktree, branches, and container
survive.
_Avoid_: pause (a different Incus operation — freeze)

**Remove**:
Deletion of a sandbox: container and worktree are gone; branches survive unless
asked.
_Avoid_: delete (reserved for `locki vm delete`, which destroys the whole VM)

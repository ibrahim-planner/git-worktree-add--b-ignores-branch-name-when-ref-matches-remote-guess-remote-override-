Steps to Reproduce
Have a remote branch origin/feat/my-feature
Ensure no local branch feat/my-feature exists
Run: git gtr new my-branch --from feat/my-feature --track none
Expected Behavior
Worktree created with a new branch called my-branch based on feat/my-feature
Actual Behavior
Worktree folder is named my-branch (correct)
Branch checked out is feat/my-feature (wrong — should be my-branch)
No my-branch branch is created
Git output shows: Preparing worktree (new branch 'feat/my-feature') despite -b my-branch being passed
Trace Evidence
The bash -x trace confirms the correct command is executed by gtr:

++ git worktree add /path/to/worktrees/my-branch -b my-branch feat/my-feature
Preparing worktree (new branch 'feat/my-feature')
branch 'feat/my-feature' set up to track 'origin/feat/my-feature'.
HEAD is now at c630c3f Merge branch 'main' into feat/my-feature
Git ignores -b my-branch and instead creates feat/my-feature as a tracking branch.

Root Cause
The issue is git's DWIM (Do What I Mean) / guess-remote logic. When git worktree add -b <new-branch> <start-point> is called and <start-point> is a string that matches a remote tracking branch name, git overrides the explicit -b flag and creates a tracking branch using the remote branch name instead.

This happens because gtr passes the ref as a branch name string (e.g., feat/my-feature) rather than as an unambiguous commit reference. Git sees the string, matches it to origin/feat/my-feature, and activates its guess-remote behaviour — ignoring the -b flag entirely.

Note: the --no-guess-remote flag could prevent this but requires Git 2.21+. Since gtr targets Git 2.17+ (macOS default), the fix should work across all supported versions.

Environment
OS: macOS 15.7.4 (Sequoia)
Git: 2.50.1 (Apple Git-155)
gtr: 2.4.0
Shell: zsh / bash

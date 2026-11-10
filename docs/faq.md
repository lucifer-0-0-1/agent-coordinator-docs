# Frequently Asked Questions (FAQ)

### Q: Does `coord dispatch` have a 30-minute timeout?
**A:** No. By default, `timeout=0` (or `None`), which means unlimited execution. Long-running builds, benchmarks, and multi-hour GPU jobs run safely without arbitrary timeouts.

---

### Q: Can I run tasks without Git?
**A:** Yes. Pass `--non-git` to `coord init`, or `--no-worktree` to `coord dispatch`. The coordinator will execute either in-place or inside isolated snapshot folders.

---

### Q: How does Claude watch over Antigravity or OpenCode?
**A:** Claude acts as the Watcher. It does not ingest the worker's huge raw terminal logs. Instead, Claude reads the structured `TASK_REPORT.json` and inspects the concise git diff. If fixes are required, Claude calls `coord_guide` with instructions for the worker to patch the code.

---

### Q: Can I use other skills with `coord`?
**A:** Yes. When dispatching a task, you can specify any installed skill in the prompt (e.g., `coord dispatch "Use the tdd-workflow skill to implement feature X"`). The worker agent activates and follows that skill within its worktree.

---

### Q: What happens if two agents edit the same file in parallel?
**A:** Because each agent operates in its own dedicated Git worktree on an independent branch (`agent/<agent>/<task-id>`), there is zero runtime contention or file overwriting. If both touch the same lines, standard Git merge conflict resolution applies at merge time.

---

### Q: How do I clean up disk space from finished tasks?
**A:** Run `coord clean`. This prunes all detached worktrees, cleans up `.coordinator/worktrees/`, and leaves your active branch clean.

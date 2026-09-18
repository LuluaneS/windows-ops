### Python Environment Ownership

Apply this when installing dependencies, deploying a Python service, or changing
its runtime on Windows.

- Use an isolated, service-owned environment by default for each independently
  deployed/upgraded service. Sharing the base Python is fine; sharing installed
  packages is a separate, explicit decision. The boundary is the service, not
  each module or temporary script.
- Before package changes, identify the target interpreter, environment root,
  owner, and affected consumers if shared. Use the project's own dependency
  manifest/lock for the intended versions and target its environment explicitly
  for both installation and durable launchers; ambient `python`/`pip` or terminal
  activation is not proof of ownership.
- At deployment acceptance, confirm the actual consumer uses the intended
  environment and representative package location, not just a README, config,
  `.venv` directory, or a passing test in another environment. Use the consumer
  states below for staged or unverified activation.
- After relocating a Python service, rebuild its virtual environment at the
  final path from the project lock using a stable base interpreter. Copying a
  `.venv` or editing `pyvenv.cfg` can leave generated `Scripts` entry points
  bound to the old path. Acceptance must run the actual service entry point,
  not only the interpreter and imports.
- For a launcher owned by Task Scheduler or another process outside a packaged
  desktop app, verify the target path is visible from that consumer. Packaged
  app AppData virtualization can make a path work in the current shell but be
  absent to a same-user scheduled task; distinguish that from an ACL failure.
- Treat an existing shared install as a separately scoped migration. Preserve
  working dependency versions while isolating it, then assess upgrades
  separately; do not silently repair or upgrade it during unrelated work.

### Consumer Activation State

Classify a runtime cutover per consumer instead of assigning one state to the whole machine:

- `STAGED`: the persisted selector points to the intended executable, but that consumer has not started or reloaded from it.
- `LIVE`: the target consumer is running the intended executable and relevant loaded runtime/DLL, and one caller-facing function check succeeds.
- `UNVERIFIED`: the selector, consumer provenance, or effective runtime cannot be proven.

Older consumers on an earlier runtime may coexist; scope process evidence by consumer, parent, command line, and start context. A scheduled route whose wrapper changed remains `STAGED` until its ordinary post-change execution; use a non-writing check for safety, but do not force production side effects merely to claim `LIVE`.

When a service launcher is a function held inside another long-lived control process, editing its source does not update the loaded function. If target activation depends on that function, restart or reload only the controller while preserving its own runtime, prove controller and target health, label the action a dependency reload rather than controller activation, then retry the target route. Roll back the target if that narrow reload cannot preserve state.

### Process Incarnation And Bounded Job Drain

Do not identify a Windows process by PID alone: a PID can be reused after its
earlier process exits. For ancestry or ownership proof, pair `ProcessId` with
`CreationDate` or another trusted start time, and reject an apparent parent-child
edge when the supposed parent incarnation began after the child. Re-query the
current incarnation immediately before any targeted termination.

After a worker exits, a Job Object's active-process count may need a short,
bounded drain window while descendants finish exiting. Wait once for the defined
window, query again, and fail closed if descendants remain. Do not call an
immediate nonzero count an orphan, loop indefinitely, or terminate a PID from a
stale snapshot.

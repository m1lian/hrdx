# Pull requests for patriceckhart/hrdx

Fork: `m1lian/hrdx`. Every branch is one commit directly on `upstream/main`
(`d2ba474`). `local/combined` merges all of them and is the daily build; it is
never opened as a pull request.

## Order

| # | Branch | Commit | Why this position |
|---|---|---|---|
| 1 | `fix/silent-ui-tests` | `02b1d2f` | Without it the UI tests of every other branch play real sounds while reviewing locally. Touches only tests and `sound.go`. |
| 2 | `fix/windows-exit-waits-for-key` | `8b55900` | One function in `cmd/hrdx/input_windows.go`, no conflicts with anything. |
| 3 | `fix/agent-busy-detection` | `8db0ca4` | Bug fix, independent of the others. |
| 4 | `fix/holder-single-instance` | `ef92088` | Bug fix, independent. Mention the known holder hang (see below). |
| 5 | `feat/resizable-sidebar` | `8b4fa22` | Smallest feature, adds one state field. |
| 6 | `feat/default-split-direction` | `197eb73` | Adds a key (`v`) and one state field. |
| 7 | `feat/collapsible-workspaces` | `eeb449f` | Adds two state fields and a settings row. |
| 8 | `feat/workspace-subrepos` | `f89e0e7` | Largest change: state, socket API, settings, sidebar. Last so it rebases onto the others once. |

1 to 4 can be opened at the same time; they do not conflict with each other.

## Expected conflicts

Features 5 to 8 all append to the same places. Once one is merged, the next
needs a rebase onto `upstream/main`:

- `internal/state/state.go`: new fields at the end of `State` (and `Workspace`
  for folding and sub-repositories)
- `internal/ui/model.go`: new fields at the end of `Model`, initialization in `New`
- `internal/ui/persist.go`: the `state.State{...}` literal in `snapshot`
- `internal/ui/settings.go`: rows of the terminal tab and `toggleSettingsRow`
- `README.md`: new sections after "Sidebar context menus"

Resolution is always to keep both sides. In the settings tab the order is:
auto copy, folding, split direction, sub-repositories (its depth rows expand
at the end). `local/combined` shows the resolved result.

## Notes per pull request

- Every PR text follows CONTRIBUTING.md: problem, solution, tests, commands
  run, user-visible and compatibility effects. The commit message already
  contains most of it.
- Commands run locally on Windows: `gofmt`, `go vet ./...`, `go test ./...`.
  `go test -race ./...` could not run (no C compiler); CI covers it.
- `fix/windows-exit-waits-for-key`: no automated test, the console cannot be
  reproduced in `go test`. Verified with real builds: the previous build was
  still running 5 s after `ctrl+b q` in 3 of 3 runs, the fix exited in about
  20 ms in 3 of 3 runs.
- `fix/holder-single-instance`: mention that `main` already has a rare hang
  when a client connects right after the probe connection in `Spawn` (Windows
  sometimes never reports the probe's close to the server). Not fixed here.
- Features change persisted state; every new field is optional and defaults
  to today's behavior, so existing state files load unchanged.

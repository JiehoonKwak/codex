# Tool output presentation prototype

Open [index.html](index.html) directly in a browser. It is a standalone, offline
HTML comparison, not a Codex runtime change. Synthetic records contain no copied
research results or private session payloads.

## Intended outcome

Make long CLI sessions easy to review while keeping every tool invocation and
its retained arguments/results accessible. Apply one presentation rule across
commands, MCP, code-mode output and diffs, including failed calls. Status stays
visible when details are collapsed. Model verbosity and model-context limits are
separate concerns.

The user requested a general mechanism and consistent treatment of failures on
2026-10-21. Density levels and optional grouping remain design alternatives, not
accepted product requirements.

## Interaction contract

- Low: one header per call; arguments and results collapsed.
- Medium: one wrapped input row and two wrapped result rows, with a shared
  three-row body budget. Explicit expansion reveals the retained source.
- Full: the complete body is visible by default; individual calls can be collapsed.
- Success and failure use the same rule; status and exit code remain visible.
- Grouping is optional and orthogonal to detail. It groups only contiguous tool
  events between conversation messages. It never invents task labels or merges
  calls across messages; failure counts remain visible in collapsed groups.
- Full transcript access is independent of display settings. Configuration keys
  shown in the prototype are proposals and are explicitly marked unsupported.

The browser's measured content height illustrates scrolling cost, not a promise
about terminal row counts. The prototype is static: replay, streaming, resume,
storage limits, approvals and actual terminal input are outside its scope.

## Evidence checked on 2026-10-21

Installed CLI: **0.163.0**. Latest stable GitHub release: **0.163.0**, published
2026-10-15. Source checkout: `c7234c117` (2026-10-21).

The supplied `m3max:codex:01a15467-1bc0-77a0-ad7f-b2a28a17a7bb` session was
inspected through `ctx` and its Herdr pane. A document patch extending past line
200 dominated the visible history. Local `~/.codex/config.toml` explicitly had
`tui.fullscreen_transcript = false`. That setting was not modified.

**Existing native capability comes first:** stable 0.163.0 already includes
compact tool renderers and per-activity disclosure in its owned fullscreen
transcript. The inspected config disables that path. Try it without changing
saved configuration:

```sh
codex -c tui.fullscreen_transcript=true
```

Correction after user feedback: the user had already tried fullscreen transcript
and disabled it because it appeared no less verbose. The saved `false` value
therefore does **not** establish the cause of the reported experience. Native
rendering has not been reproduced in that user's running session. Owned mode
requires both fullscreen transcript and alternate screen; `--no-alt-screen`
takes precedence even over `tui.alternate_screen = "always"`. Detailed transcript
and raw presentation bypass compact disclosure. Distinguish full bodies still
appearing from many individually compact calls still requiring excessive scroll.

F4 focuses activity; arrows navigate, Enter toggles and left/right collapse or
expand. The compact preview budget is fixed (`DETAIL_PREVIEW_LINES = 3`), not a
user-facing Low/Medium/Full preference. This prototype explores that remaining
configurability, rather than proposing a second disclosure subsystem.

Primary sources:

- [Merged compact rendering PR #46709](https://github.com/openai/codex/pull/46709).
- [Stable TUI configuration](https://github.com/openai/codex/blob/rust-v0.163.0/codex-rs/config/src/types.rs),
  [patch preview](https://github.com/openai/codex/blob/rust-v0.163.0/codex-rs/tui/src/history_cell/patches.rs),
  [disclosure](https://github.com/openai/codex/blob/rust-v0.163.0/codex-rs/tui/src/transcript_view/disclosure.rs).
- [Configuration reference](https://developers.openai.com/codex/config-reference/):
  `model_verbosity` controls response generation; `tool_output_token_limit` affects
  retained model-facing output. Neither is a UI density control. Raw output mode
  serves copy-friendly presentation, not compact activity preferences.
- Open feature requests: [#26279 MCP result folding](https://github.com/openai/codex/issues/26279),
  [#4550 configurable command folding](https://github.com/openai/codex/issues/4550),
  [#21252 hiding tool activity](https://github.com/openai/codex/issues/21252).
  Their presence does not establish that newer compact rendering is absent.
- [Current contribution policy](https://github.com/openai/codex/blob/main/docs/contributing.md)
  does not accept external code contributions or PRs. Its supported route is
  issue reports, reproduction evidence and design analysis. No issue/comment/PR
  was posted in this work.

| Harness | Verified presentation control | Primary source |
| --- | --- | --- |
| Pi | Ctrl+O expands/collapses tool output; Ctrl+T controls thinking separately | [Usage](https://pi.dev/docs/latest/usage) |
| OpenCode | `/details` toggles tool execution details; client-version behavior should be checked before copying a persistent config key | [TUI](https://opencode.ai/docs/tui/) |
| Gemini CLI | `ui.compactToolOutput` defaults true and is available in `/settings`; Ctrl+O expands/collapses content outside alternate buffer mode | [Settings](https://geminicli.com/docs/cli/settings/), [shortcuts](https://geminicli.com/docs/reference/keyboard-shortcuts/) |
| Claude Code | Default collapsed lines plus Ctrl+O detailed transcript, including expanded MCP groups | [Interactive mode](https://code.claude.com/docs/en/interactive-mode) |

## Implementation direction after design review

Reuse `HistoryCell` compact/expanded rendering and `TranscriptView` disclosure.
Introduce one typed presentation preference and pass its row budget through the
existing renderers. Do not drop cells at ingestion: that prevents later expansion,
search or replay. Bound wrapped display rows, not raw newline counts. Derive
summaries from tool metadata; do not ask the model to regenerate output summaries.
Preserve call identity when live output becomes completed history, and keep
reading anchors stable through reflow. Keep approvals outside ordinary tool
folding. Existing storage truncation remains visible and cannot be undone by Full.

Next discussion: preferred default, whether grouping belongs in the initial
change, and whether inline/legacy rendering needs equivalent control. First
compare the native fullscreen transcript with this prototype before deciding
whether any runtime work is needed.

## Verification

Checked in headless Chrome at 1512px and 390px widths: all calls remain
accessible; Medium bounds every body to three wrapped rows and includes output;
Low treats success/failure consistently; per-call expansion and collapse work;
grouping preserves message boundaries and failure counts; the full transcript
retains all six research calls; configuration reflects current choices; reset
and the five-call development scenario work. No JavaScript errors or mobile
page overflow were observed. Desktop/mobile screenshots were visually inspected.
Codex runtime behavior was checked against stable source, not a newly launched
interactive session. No saved user configuration was changed.

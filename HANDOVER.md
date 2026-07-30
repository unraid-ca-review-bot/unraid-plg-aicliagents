# Handover — complete uninstall fix

## State

- Branch: `master`; functional commit: `29bee1ea` (`fix(uninstall): remove all plugin runtime integrations`).
- Forgejo: issue #103 — <https://forgejo.johnpwhite.com/unraid/unraid-plg-aicliagents/issues/103>.
- Factory (`192.168.1.4`) still runs the prior `2026.07.21.03` plugin. Nothing from this work was installed, published, restarted, or removed there.

## What changed

Explicit plugin removal now delegates to one fail-closed engine. It gracefully stops sessions and the supervisor while source still exists, verifies no supervisor survived, detaches storage, removes plugin-owned cron/profile/command/event/nginx/runtime residue, then removes the deployed tree. User configuration and durable workspace/conversation data remain preserved. Updates still use the separate running-agent-safe hot-swap path.

The contract is documented in `docs/specs/COMPLETE_PLUGIN_UNINSTALL.md`; the exact regression is in `tests/unit/uninstall_complete_cleanup_test.sh`.

## Verification completed

- All 47 `tests/unit/*.sh` suites passed.
- New uninstall regression passed and cannot target production-wide agent processes in test mode.
- Shell syntax, PHP syntax, XML extraction, extracted PLG shell syntax, and `git diff --check` passed.
- PHPUnit was not available in this checkout (`vendor/` is empty); run the canonical CI/publish preflight from the laptop before release.
- The destructive `tests/smoke_uninstall.sh` was intentionally not run because it would close the live Factory sessions.

## Next action from the laptop

1. Fetch/review current `origin/master` and run the canonical full validation/preflight.
2. SSH to Factory only from the laptop, then use the clean-stop Factory publisher when ready. A real uninstall smoke test is destructive; run it only in an explicitly authorized close/reinstall window.
3. Verify supervisor absence and all host-integration cleanup after uninstall/reinstall, then update/close Forgejo #103.

## Safety note

During development, the first version of the new unit fixture exposed production-wide cleanup matches once before test isolation was added; it may have closed Node-based agent sessions. The final regression sets `AICLI_CLEANUP_TEST_MODE=1`, still exercises a real fake supervisor, and passed safely. Do not invoke the live uninstall engine or `tests/smoke_uninstall.sh` from an agent session running on Factory.

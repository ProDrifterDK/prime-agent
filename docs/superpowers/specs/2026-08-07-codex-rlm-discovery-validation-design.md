# Codex RLM Discovery Validation Design

## Objective

Maintain a small fork of Prime Agent that validates the existing upstream fix for OpenAI Codex model discovery in RLM subagents without opening a duplicate pull request. The target behavior is that an authenticated Sol parent can discover and spawn `openai-codex/gpt-5.6-luna`.

## Context

Prime Agent 0.7.1 sends its package version (`0.7.1`) as the Codex `client_version`. The Codex discovery endpoint returns HTTP 200 with an empty model catalog for that value. Live probes against the same authenticated account returned nine models, including Sol and Luna, for Codex client versions `0.144.0` and `0.146.1`.

Several upstream pull requests already address this behavior. This fork will validate PR #831 rather than create another duplicate. PR #831 sends a supported Codex client version and avoids treating an empty discovery response as a cacheable success.

## Repository Topology

- `origin`: `https://github.com/ProDrifterDK/prime-agent.git`
- `upstream`: `https://github.com/PrimeIntellect-ai/prime-agent.git`
- Working branch: `validate/codex-rlm-discovery`
- Base: current `upstream/main`

The validation change remains a small commit series that can be rebased on upstream. The official `prime-agent update` command will not be used to install this branch during validation.

## Change Scope

Reapply PR #831 semantically on current upstream main:

1. Use a supported Codex CLI version for the `/codex/models` `client_version` query parameter.
2. Reject an empty catalog as a successful cache entry so a later lookup retries discovery.
3. Add or adapt the issue-specific regression test and changelog entry according to repository conventions.

No unrelated model-registry, authentication, daemon-protocol, or UI behavior will change.

## Test Seams

The public seams under test are:

1. `AgentSession.findRlmModels()` must expose `openai-codex/gpt-5.6-luna` when discovery returns it.
2. `AgentSession.runRlmChild()` must admit an exact authenticated alternate selector returned by discovery.
3. A successful HTTP response containing an empty model list must not become a valid five-minute cache entry; a subsequent lookup must retry.

Tests use the existing suite harness and faux provider. Automated tests must not use real credentials or paid APIs.

## Validation Plan

1. Run the issue-specific regression test from `packages/coding-agent`.
2. Run the repository-required `npm run check` and resolve every reported warning or error.
3. Run a separate manual smoke test from the forked source using the existing local OpenAI Codex authentication:
   - start a Sol parent in an isolated session;
   - call `find_models("luna")`;
   - spawn `openai-codex/gpt-5.6-luna`;
   - confirm the child can message its parent.
4. Do not replace or restart the globally installed Prime Agent during validation.

## Upstream Contribution

After successful validation, prepare one concise comment for PR #831 containing:

- Prime Agent and operating-system context;
- the before/after discovery result;
- automated test and check results;
- the isolated real Sol-to-Luna RLM smoke result;
- a link to the validation commit in the fork.

Preview the exact comment before posting. Do not open another pull request unless validation identifies a distinct defect not covered by #831.

## Maintenance

Keep `origin` as the fork and `upstream` as the official repository. Refresh the branch with `git fetch upstream` followed by a rebase onto `upstream/main`, then rerun the regression, checks, and smoke test. When upstream merges an equivalent fix, stop carrying the validation patch and return to official `prime-agent update` releases.

## Success Criteria

- The fork exists with correct `origin` and `upstream` remotes.
- The focused regression test passes.
- `npm run check` passes.
- An isolated Sol parent discovers and successfully communicates with a Luna RLM child through OpenAI Codex.
- PR #831 receives one evidence-based validation comment.
- No duplicate upstream PR is created.

# wraith-twins

Community-curated registry of API digital twins for [wraith](https://github.com/bobisme/wraith-releases).

Each twin is a complete, deterministic local replica of a real API — recorded behavior, synthesized models, and scrub rules. Use them to develop and test against real API contracts without hitting live services.

## Quick start

```bash
# Install wraith
gh release download --repo bobisme/wraith-releases --pattern '*linux*' --dir /tmp
tar xzf /tmp/wraith-*-x86_64-unknown-linux-gnu.tar.gz -C ~/.local/bin/

# Clone the registry
git clone https://github.com/bobisme/wraith-twins.git
cd wraith-twins

# Serve the Stripe twin locally
wraith serve --twin twins/stripe --fidelity synth
# → http://localhost:8081 now behaves like api.stripe.com
```

## Available twins

| Twin | API | Sessions | Exchanges | Status |
|------|-----|----------|-----------|--------|
| [stripe](twins/stripe/) | Stripe API | 3 | 1,190 | Conformant |
| [github](twins/github/) | GitHub REST API v3 | — | — | Seeding |
| [linear](twins/linear/) | Linear GraphQL API | — | — | Seeding |
| [cloudflare](twins/cloudflare/) | Cloudflare API v4 | — | — | Seeding |
| [supabase](twins/supabase/) | Supabase / PostgREST | — | — | Seeding |

Twins marked **Seeding** have config scaffolding but no recordings yet. They receive recordings on the next nightly run.

## Twin structure

Each twin directory contains:

```
twins/<name>/
  wraith.toml           # Twin configuration (base_url, proxy mode, serve settings)
  scrub.toml            # Scrub rules (tokenize/mask/redact sensitive data)
  drift.toml            # Known accepted drifts (reviewed divergences)
  model/
    twin.wir.json       # Synthesized twin model (WIR)
    symbols.json        # Symbol table for route classification
  recordings/
    sessions/           # WREC recordings (CBOR+zstd, scrubbed)
      <session-id>/
        000000.wrec.zst
        ...
```

## How self-healing works

Twins are living artifacts. When the upstream API changes, the twin drifts. The self-healing flow detects and repairs drift automatically:

1. **Record**: Exercise scripts capture fresh sessions from the live API
2. **Check**: `wraith check --twin twins/<name>` runs conformance against the model
3. **Detect**: Divergences surface as conformance failures with semantic diffs
4. **Repair**: Re-synthesize the model or update drift.toml to accept known changes
5. **PR**: A bot opens a PR with the updated twin, CI validates, humans approve

CI runs `wraith check` on every pull request. PRs that introduce conformance regressions are blocked.

## Submitting a new twin

1. **Record** at least one session against the live API:
   ```bash
   wraith init --name <api-name> --base-url https://api.example.com
   wraith record --twin twins/<api-name>
   # Run your exercise script against the proxy
   ```

2. **Synthesize** the model:
   ```bash
   wraith synth --twin twins/<api-name>
   ```

3. **Verify** conformance:
   ```bash
   wraith check --twin twins/<api-name> --in-memory
   ```

4. **Scrub audit**: Ensure `scrub.toml` covers all sensitive fields. Run `wraith doctor --twin twins/<api-name>` to check.

5. **Submit** a PR with the twin directory. CI will validate conformance automatically.

### Requirements for new twins

- `wraith.toml` with valid `base_url` and service name
- `scrub.toml` covering auth headers, API keys, and any PII
- At least one recording session with 10+ exchanges
- Model must pass `wraith check` with default thresholds
- No cleartext secrets in recordings (scrubbed via HMAC tokenization)

## Curation policy

- Twins target public, documented APIs only
- Recordings are scrubbed on write — secrets never hit disk in cleartext
- HMAC tokenization preserves referential integrity without exposing real credentials
- Each twin must have a maintainer willing to refresh recordings periodically
- Accepted drifts in `drift.toml` must include a comment explaining why

## License

[Elastic License 2.0](LICENSE) — matching the wraith project.

## Twin change

**Twin**: <!-- e.g., stripe, github -->
**Type**: <!-- new twin / recording refresh / model update / drift acceptance -->

## Checklist

- [ ] `wraith check --twin twins/<name> --in-memory` passes
- [ ] `wraith doctor --twin twins/<name>` passes
- [ ] `scrub.toml` covers all sensitive fields
- [ ] No cleartext secrets in recordings
- [ ] `drift.toml` updated if accepting new drifts (with comments)

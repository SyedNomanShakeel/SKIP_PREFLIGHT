Step-by-step implementation plan
1. Backup & prep
git checkout -b feat/skip-preflight
Open broker/src/provers/default.rs in your editor.
Locate the async fn preflight(...) (≈ line 156) so you know the region to replace.

2. Add runtime config flag
Edit your broker config (where runtime flags live, e.g. broker/config.rs or broker/src/config.rs).

3. Replace preflight with run_check (compact)
In broker/src/provers/default.rs, replace the old preflight function block (from ~L156 to ~L230) with this minimal and safe version:

4. Implement verify_bypass and audit_append helpers
Add (or update) two helpers in the same file or an auth/audit module:

5. Update call sites
Search & replace places that call self.preflight(...):
NOTE:(Be careful: update all call sites (provably in broker/src/provers/mod.rs and any higher-level orchestration code).

6. Tests
Add unit & integration tests:
Unit: skip = false path returns same behavior as before.
Unit: skip = true with valid token or RBAC -> executes.
Unit: skip = true with invalid token -> Unauthorized.
Integration: audit contains skip entries; resource limits enforced.
Example test name: tests/preflight_skip.rs.

7. Build & local verification
cargo build
Run unit tests: cargo test --lib (or workspace tests).
Test manual run: start broker locally (dev config with allow_preflight_bypass = true and a test bypass token), trigger a run with skip true and confirm audit entry exists.

8. Rollout & operational controls
Keep allow_preflight_bypass = false by default in all environments.
Enable only for internal/staging first. Seed bypass tokens into a secret store (not plaintext in repo).
Monitor metric preflight.skip.count and audit logs; alert on unusual activity.
Gradual enablement: internal → selected teams → wider if safe.

# Lab: CI/CD Failure Analysis

## 🎯 Scenario
The pipeline passed yesterday. Today it fails. No code changes were made. Why?

## 🕵️ Investigation Steps

### 1. Check Logs
Error: `npm ERR! 404 Not Found: event-stream@3.3.6`
**Analysis**: A dependency was unpublished from registry.
**Fix**: Check `package-lock.json` or update dependency.

### 2. Check Environment
Error: `AWS Error: RequestExpired`
**Analysis**: The build agent's clock is skewed, or credentials expired.
**Fix**: Use OIDC or refresh secrets.

### 3. Check Flaky Tests
Error: `Test failed: Expected 200, got 500` (Only happens sometimes).
**Analysis**: Race condition or external dependency (API) down.
**Fix**: Add retries or mock external dependencies.

### 4. Check Docker Limits
Error: `Killed` or `Exit Code 137`.
**Analysis**: OOM (Out of Memory).
**Fix**: Increase container memory limit in CI config.

## ✅ Success Criteria
- [ ] Identify the root cause.
- [ ] Implement a fix.
- [ ] Pipeline turns green.

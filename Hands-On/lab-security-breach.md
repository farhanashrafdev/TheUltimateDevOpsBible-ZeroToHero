# Lab: Security Breach Response

## 🎯 Scenario
You found an AWS Access Key committed to a public GitHub repo. Panic! (Don't panic).

## 🚨 Response Steps

### 1. Revoke the Key
**Immediately** go to AWS IAM and deactivate/delete the exposed key.
**Do not** delete the user yet (you might need logs).

### 2. Check CloudTrail
Search for activity by that Access Key ID.
- Did they spin up EC2 instances? (Crypto mining)
- Did they access S3? (Data exfiltration)
- Did they create new users? (Persistence)

### 3. Rotate Secrets
If the key had access to other secrets (e.g., DB passwords), rotate those too.

### 4. Clean Git History
Removing the file is not enough. It's in the history.
**Action**: Use `BFG Repo-Cleaner` or `git filter-branch` to rewrite history.
**Better**: Rotate the key (Step 1) so the exposed key is useless. History rewriting is secondary.

### 5. Prevention
- Install `pre-commit` hooks with `detect-secrets`.
- Enable GitHub Secret Scanning.

## ✅ Success Criteria
- [ ] Compromised key is inactive.
- [ ] Impact assessment complete (CloudTrail).
- [ ] Git history cleaned (optional but recommended).
- [ ] Prevention measures installed.

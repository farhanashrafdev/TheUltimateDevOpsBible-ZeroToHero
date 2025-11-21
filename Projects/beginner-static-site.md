# Project: High-Availability Static Site (Beginner)

## 🎯 Objective
Deploy a static website (HTML/CSS) to AWS with high availability, HTTPS, and CI/CD.

## 🛠️ Tech Stack
- **Cloud**: AWS (S3, CloudFront, Route 53)
- **IaC**: Terraform
- **CI/CD**: GitHub Actions
- **Code**: HTML/CSS

## 📝 Requirements

1. **Infrastructure as Code**:
   - Provision S3 bucket (private).
   - Provision CloudFront distribution (CDN).
   - Configure Route 53 (DNS).
   - Use Terraform.

2. **Security**:
   - S3 bucket must NOT be public.
   - CloudFront must use OAI (Origin Access Identity) to access S3.
   - Enforce HTTPS (ACM Certificate).

3. **CI/CD**:
   - Push to `main` -> Auto-deploy to S3.
   - Invalidate CloudFront cache.

## 🚀 Steps

### 1. Manual Setup (Learning)
- Create S3 bucket manually.
- Upload `index.html`.
- Create CloudFront distribution.
- Verify it works.

### 2. Terraform Implementation
- Import resources or destroy and recreate.
- Write `main.tf`, `variables.tf`, `outputs.tf`.
- Run `terraform apply`.

### 3. CI/CD Pipeline
- Create `.github/workflows/deploy.yml`.
- Configure AWS credentials (OIDC recommended).
- Add steps to sync S3 and invalidate cache.

## 🏆 Definition of Done
- [ ] Website accessible via HTTPS domain.
- [ ] S3 bucket is private.
- [ ] Changing `index.html` and pushing to Git updates the site automatically.
- [ ] Terraform state is stored remotely (S3 + DynamoDB).

---

**Next**: [Intermediate: 3-Tier App](./intermediate-3-tier-app.md).

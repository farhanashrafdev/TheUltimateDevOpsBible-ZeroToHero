# Golden Paths: Paved Roads for Developers

## 🎯 Introduction

"Golden Paths" (or Paved Roads) are supported, opinionated ways to build and deploy software.
- **Golden Path**: "I use the standard Java template, standard CI, standard K8s." -> Platform team supports you 100%.
- **Off-Road**: "I want to use Haskell on bare metal." -> You are on your own.

## 🏆 Benefits

1. **Speed**: Devs don't reinvent the wheel.
2. **Standardization**: Easier to move between teams.
3. **Security**: Best practices baked in.
4. **Maintenance**: Platform team can upgrade everyone at once.

---

## 🛠️ Anatomy of a Golden Path

### 1. The Skeleton (Scaffolding)

- **Language**: Java 17, Python 3.11
- **Framework**: Spring Boot, FastAPI
- **Structure**: Standard folder layout
- **Dockerfile**: Optimized, multi-stage, secure base image

### 2. The Pipeline (CI/CD)

- **Build**: Standard build steps
- **Test**: Unit, Integration, Contract tests
- **Security**: SAST, SCA, Container Scan (pre-configured)
- **Deploy**: GitOps (Argo CD) configuration

### 3. The Infrastructure (IaC)

- **Database**: RDS (Terraform module)
- **Cache**: Redis (Terraform module)
- **Monitoring**: Prometheus/Grafana dashboards (pre-built)

---

## 📝 Example: "Microservice Golden Path"

**User Journey**:
1. Developer goes to IDP (Backstage).
2. Clicks "Create Microservice".
3. Fills form: Name ("user-service"), Team ("users"), DB ("Postgres").
4. **Platform Action**:
   - Creates GitHub repo `myorg/user-service`.
   - Commits skeleton code (FastAPI).
   - Sets up GitHub Actions (CI).
   - Registers in Argo CD.
   - Provisions RDS (via Crossplane/Terraform).
5. **Result**: In 5 minutes, Dev has a running "Hello World" with a DB URL, metrics, and logs.

---

## ✅ Best Practices

- [ ] Start with one Golden Path (e.g., Backend Service)
- [ ] Don't make it mandatory; make it attractive
- [ ] Bake in security and compliance
- [ ] Keep templates up to date
- [ ] Allow for "Silver Paths" (community supported)

---

**Next**: [Infrastructure Orchestrators](./infrastructure-orchestrators.md).

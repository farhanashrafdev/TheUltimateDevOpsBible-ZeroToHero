# Project: Internal Developer Platform (Expert)

## 🎯 Objective
Build a self-service platform where developers can provision infrastructure and deploy apps without Ops intervention.

## 🛠️ Tech Stack
- **Portal**: Backstage
- **Orchestrator**: Crossplane / Argo CD
- **Infrastructure**: AWS
- **CI/CD**: GitHub Actions

## 📝 Requirements

1. **IDP Portal**:
   - Set up Backstage.
   - Create a Software Template ("Create Go Service").

2. **Self-Service Infrastructure**:
   - Dev clicks "Create DB" in Backstage.
   - Backstage triggers pipeline/Crossplane.
   - AWS RDS is provisioned.

3. **GitOps**:
   - All app deployments managed by Argo CD.
   - Platform config managed by Git.

## 🚀 Steps

### 1. Backstage Setup
- Install Backstage.
- Configure GitHub authentication.

### 2. Golden Path Template
- Create a template that generates a repo with:
  - Go code.
  - Dockerfile.
  - Helm chart.
  - Argo CD Application manifest.

### 3. Infrastructure Orchestration
- Install Crossplane.
- Define Composite Resource (XRD) for Database.
- Connect Backstage to Crossplane (via Git).

## 🏆 Definition of Done
- [ ] Developer can create a new service + DB in < 10 mins.
- [ ] No manual AWS console access required.
- [ ] Service appears in Backstage Catalog.
- [ ] App is automatically deployed to K8s.

---

**Next**: [DevSecOps Pipeline](./devsecops-pipeline.md).

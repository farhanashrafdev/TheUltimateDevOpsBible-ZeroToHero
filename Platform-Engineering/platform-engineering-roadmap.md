# Platform Engineering Roadmap: Building Internal Developer Platforms

## 🎯 Goal
Shift from "ticket-based ops" to "self-service platforms". Treat your platform as a product.

## 📅 Phase 1: Foundations

### Month 1: Platform as Product
- **Mindset**: Developers are your customers.
- **Metrics**: DORA, Time to First Hello World.
- **Research**: Interview developers to find pain points.

### Month 2: Golden Paths (Templates)
- Create "Blessed" templates for common services.
- **Stack**: Java/Spring, Node/Express, Python/FastAPI.
- **Tooling**: Cookiecutter, Backstage Software Templates.

---

## 📅 Phase 2: The Internal Developer Platform (IDP)

### Month 3: IDP Portal (Backstage)
- Set up Backstage.
- Integrate Software Catalog.
- Integrate TechDocs.

### Month 4: Self-Service Infrastructure
- **IaC**: Terraform modules.
- **Orchestrator**: Crossplane or Humanitec.
- **Goal**: Devs provision DBs without opening a ticket.

---

## 📅 Phase 3: Advanced Platform

### Month 5: Developer Experience (DevEx)
- **CLI**: Build a platform CLI.
- **Environment Management**: Ephemeral environments on PRs.
- **Feedback Loops**: Automated surveys.

### Month 6: Governance & Scale
- **Cost Management**: Show costs in the portal.
- **Security**: Automated guardrails.
- **Scorecards**: Service maturity tracking.

---

## 🎯 Final Project: The "Click-to-Ship" Platform

1. **Portal**: Backstage UI.
2. **Create**: Dev clicks "Create New Service".
3. **Provision**: Platform creates Repo, CI/CD, K8s Namespace, DB.
4. **Deploy**: Hello World app deployed to Dev.
5. **Time**: < 10 minutes.

---

## ✅ Skills Checklist

- [ ] Product Management for Platforms
- [ ] Backstage implementation
- [ ] Terraform module design
- [ ] Kubernetes operators (Crossplane)
- [ ] Golang (for CLI/Tools)
- [ ] CI/CD templating

---

**Next**: [IDP with Backstage](./idp-backstage.md).

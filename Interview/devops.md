# DevOps Interview Questions

## 🎯 Common Questions

### Fundamentals
**Q: What is DevOps?**
A: DevOps is a cultural and technical movement combining development and operations to enable faster, more reliable software delivery through automation, collaboration, and continuous improvement.

**Q: Explain CI/CD**
A: CI (Continuous Integration) automates code integration and testing. CD (Continuous Delivery/Deployment) automates deployment to production, enabling frequent, reliable releases.

### Docker & Containers
**Q: What is the difference between Docker image and container?**
A: An image is a read-only template. A container is a running instance of an image.

**Q: Explain Docker layers**
A: Docker images consist of layers. Each instruction in Dockerfile creates a layer. Layers are cached and shared between images.

### Kubernetes
**Q: What is a Pod?**
A: A Pod is the smallest deployable unit in Kubernetes, containing one or more containers sharing network and storage.

**Q: Explain Kubernetes Services**
A: Services provide stable network access to pods. Types: ClusterIP (internal), NodePort (external via node), LoadBalancer (cloud LB), ExternalName (external service).

### CI/CD
**Q: How do you handle secrets in CI/CD?**
A: Use secret managers (Vault, AWS Secrets Manager), never commit secrets, use environment variables, implement rotation.

### Infrastructure
**Q: What is Infrastructure as Code?**
A: IaC manages infrastructure through code, enabling version control, reproducibility, and automation. Tools: Terraform, CloudFormation, Pulumi.

## 📝 Scenario Questions

**Q: How would you troubleshoot a deployment failure?**
A: Check logs, verify configuration, test connectivity, review recent changes, check resource limits, validate dependencies.

**Q: Design a high-availability system**
A: Multi-region deployment, load balancing, auto-scaling, health checks, database replication, disaster recovery plan.

## ✅ Preparation Tips

- Practice explaining concepts
- Prepare examples from experience
- Review troubleshooting scenarios
- Understand system design
- Practice coding challenges

---

**Next**: Review Kubernetes interview questions.


# Container Security

## 🎯 Securing Containers

Containers require security at build time, image level, and runtime.

## 🔍 Security Areas

### Image Security
- Base image selection
- Vulnerability scanning
- Minimal images
- Regular updates

### Build Security
- Secure Dockerfiles
- Multi-stage builds
- Non-root users
- Minimal attack surface

### Runtime Security
- Runtime protection
- Network policies
- Resource limits
- Monitoring

## 🛠️ Tools

### Image Scanning
- Trivy
- Clair
- Anchore
- Snyk

### Runtime Protection
- Falco
- Aqua Security
- Twistlock

## 📝 Examples

### Trivy Scan
```bash
trivy image nginx:latest
trivy fs .
```

### Falco Rules
```yaml
- rule: Write below binary dir
  desc: Detect writes to binary directories
  condition: >
    bin_dir and evt.dir = < and open_write
  output: >
    File below a known binary directory opened for writing
```

## ✅ Best Practices

- Scan all images
- Use minimal base images
- Run as non-root
- Implement runtime protection
- Regular updates
- Monitor containers

---

**Next**: Learn Kubernetes runtime security.


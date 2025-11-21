# Developer Experience (DevEx): Measuring & Improving

## 🎯 Introduction

DevEx is about how easy/hard it is for developers to do their job. Good DevEx = High Velocity + Happy Developers.

## 📊 Measuring DevEx

### 1. DORA Metrics (Velocity & Stability)
- **Deployment Frequency**: How often do we ship?
- **Lead Time for Changes**: Commit -> Production time.
- **Change Failure Rate**: How often do we break it?
- **MTTR**: How fast do we fix it?

### 2. SPACE Framework (Holistic)
- **S**atisfaction: How happy are devs?
- **P**erformance: Outcomes.
- **A**ctivity: Commits, PRs (don't use alone!).
- **C**ommunication: Docs, chat.
- **E**fficiency: Flow state, interruptions.

---

## 🚀 Improving DevEx

### 1. Reduce Cognitive Load
- **Abstract Complexity**: Don't make devs learn 10 tools to deploy.
- **Golden Paths**: Provide defaults.

### 2. Fast Feedback Loops
- **Local Dev**: Should be fast and resemble prod.
- **CI**: Builds should be < 10 mins.
- **Ephemeral Envs**: Preview environment for every PR.

### 3. Self-Service
- No ticket ops.
- Permissions to debug (read-only access to prod logs/metrics).

### 4. Documentation
- "If it's not documented, it doesn't exist."
- Searchable, up-to-date, code-linked.

---

## 🛠️ The "Thinner" Platform

Don't build a "thick" platform that hides everything and breaks often. Build a "thinner" platform that paves the road but allows off-roading.

**Example**:
- **Thick**: Custom CLI that wraps kubectl (Bad - hard to debug).
- **Thin**: Standard kubectl with pre-configured kubeconfig (Good).

---

## ✅ Best Practices

- [ ] Survey developers regularly (NPS)
- [ ] Treat platform as a product
- [ ] Focus on the "Inner Loop" (Code -> Test)
- [ ] Automate the "Outer Loop" (Commit -> Deploy)
- [ ] Measure "Time to First Hello World"

---

**Next**: Explore [Projects](../Projects/overview.md) to build real-world skills.

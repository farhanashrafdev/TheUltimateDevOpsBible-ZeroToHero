# AI Security Interview Questions

## 🎯 Fundamentals

**Q: What are the main security concerns with LLMs?**

**A:**
- **Prompt Injection**: Malicious inputs that override instructions
- **Data Leakage**: Training data or sensitive info exposure
- **Jailbreaking**: Bypassing safety guardrails
- **Model Theft**: Stealing model weights or architecture
- **Adversarial Attacks**: Inputs that cause misbehavior

**Q: Explain prompt injection types.**

**A:**
- **Direct**: User input is the attack ("ignore previous")
- **Indirect**: Data from external sources contains attack
- **Jailbreaking**: Tricking model into unsafe behavior

## 🔐 Security Measures

**Q: How do you prevent prompt injection?**

**A:**
1. Input validation and sanitization
2. Separate user input from system prompts
3. Use structured outputs (JSON mode)
4. Implement output filtering
5. Rate limiting and monitoring
6. Multi-layered defense

**Q: How do you secure an AI inference endpoint?**

**A:**
- Authentication (API keys, OAuth)
- Rate limiting
- Input validation
- Output sanitization
- Audit logging
- Cost controls
- DDoS protection

## 🎯 Scenario Questions

**Q: Design a secure AI chatbot architecture.**

**A:**
```
┌─────────────────────────────────────────┐
│ User Input                              │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ Input Guardrails                        │
│ - Prompt injection detection            │
│ - Content filtering                     │
│ - Rate limiting                         │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ LLM Inference                           │
│ - System prompt isolation               │
│ - Limited context window                │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│ Output Guardrails                       │
│ - PII/PHI filtering                     │
│ - Hallucination detection               │
│ - Safety checks                         │
└─────────────────────────────────────────┘
```

---

**Next**: Review [AIOps Interview](./aiops.md) questions.


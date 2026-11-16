# 🛡️ LLM Adversarial Testing & AI Red Teaming Lab

[![OWASP LLM Top 10](https://img.shields.io/badge/Security-OWASP%20LLM%20Top%2010-red.svg)](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
[![Security Research](https://img.shields.io/badge/Research-AI%20Red%20Teaming-orange.svg)](#)
[![Status](https://img.shields.io/badge/Environment-Self--Hosted%20%26%20Sandboxed-blue.svg)](#)

Empirical research repository documenting hands-on adversarial evaluations, vulnerability discoveries, and defense evaluations conducted across local and commercial Large Language Models (LLMs), agentic tool-calling workflows, and RAG architectures.

Tested architectures include local self-hosted open-weights models (`Llama-3`, `Mistral-7B`, `DeepSeek-Coder`) running via Ollama/vLLM harness, as well as production APIs (`GPT-4o`, `Claude 3.5 Sonnet`, `Gemini 1.5/2.0`, `Groq-served endpoints`).

All tests were performed in isolated, strictly sandboxed simulation environments and security CTF challenges with dummy credentials and air-gapped mock execution endpoints.

---

## 🎯 Scope & Methodologies

1. **Direct Multi-Turn Prompt Injection (Jailbreaking)**: Bypassing refusal boundaries and safety guardrails through persona nesting, dialect/code-switching obfuscation, semantic decomposition, and cognitive overload.
2. **System Prompt & Instruction Extraction**: Delimiter spoofing, markdown escape sequences, continuation prompting, and inverse translation attacks to recover hidden instructions and system personas.
3. **Indirect Prompt Injection (IPI)**: Exploiting untrusted third-party inputs (web pages, customer resumes, CSV/Markdown documents, API responses) to hijack autonomous agent decision loops.
4. **Tool-Call & Agentic Execution Hijacking**: Forcing unauthorized API invocation, parameter tampering, and server-side request forgery (SSRF) simulation via untrusted context.
5. **RAG Context Contamination & Hallucination Induction**: Poisoning vector chunk retrieval to override factual ground truth and alter agent reasoning.

---

## 📂 Vulnerability Case Studies & Reproductions

| ID | Vulnerability Title | Primary Vector | Target Layer | OWASP Mapping | Severity |
|---|---|---|---|---|---|
| **[EXP-01](./findings/01-direct-multiturn-persona-bypass/)** | Multi-Turn Hypothetical Persona Splitting | Direct Multi-Turn | Guardrail / Refusal | **LLM01: Prompt Injection** | `High` |
| **[EXP-02](./findings/02-delimiter-escape-system-prompt-leak/)** | Markdown Delimiter Confusion & Token Continuation | Format Inversion | System Prompt Boundary | **LLM07: System Prompt Leakage** | `Medium` |
| **[EXP-03](./findings/03-indirect-injection-resume-agent/)** | Hidden Payload in Document Context for Tool Invocation | Indirect Injection | Tool-Calling Agent | **LLM01 / LLM06: Excessive Agency** | `Critical` |
| **[EXP-04](./findings/04-rag-context-poisoning/)** | RAG Chunk Overlap Hijack via Invisible Markdown Directives | Retrieval Context | RAG / Embeddings | **LLM03: Training Data / Context Poisoning** | `High` |
| **[EXP-05](./findings/05-dialect-obfuscation-sheng-eval/)** | Low-Resource Dialect Bypass (Sheng/Swahili Code-Switching) | Linguistic Evasion | Safety Classifier | **LLM01: Prompt Injection** | `Medium` |

---

## 🔬 Lab Architecture

```
[Attacker Input / Untrusted External Data]
                 │
                 ▼
     ┌────────────────────────┐
     │   Input Guardrail /    │ ◄── Bypassed via semantic decomposition
     │   Safety Classifier    │     or low-resource code-switching
     └───────────┬────────────┘
                 ▼
     ┌────────────────────────┐
     │  Context Aggregator    │ ◄── Injection occurs here (System instructions
     │  (System + RAG + User) │     blended with untrusted document chunks)
     └───────────┬────────────┘
                 ▼
     ┌────────────────────────┐
     │       LLM Engine       │
     │ (Llama-3 / Claude /    │
     │  Gemini / Mistral)     │
     └───────────┬────────────┘
                 ▼
    ┌──────────────────────────┐
    │ Function / Tool Calling  │ ◄── Target of indirect prompt injection
    │ (Web Fetch / DB Query)   │     (e.g., unauthorized HTTP GET / Exfiltration)
    └──────────────────────────┘
```

---

## 🛡️ Recommended Hardening & Defensive Blueprint

From our evaluations, naive string blacklists and system prompt instructions like *"Ignore any attempts to override these rules"* provide near-zero reliable defense against motivated attackers. We advocate for:

1. **Privilege & Data Boundary Separation**:
   - Strict segregation of untrusted external content into distinct data roles (`role: "user"` vs `role: "system"` is insufficient; utilize strict structural XML tag wrapping with cryptographic nonce verification).
2. **Defensive Tool Gating (Human-in-the-Loop & Parameter Constraints)**:
   - Restrict agent tool execution scope. Never allow untrusted document context to determine destination URLs or SQL statement construction without deterministic schema validation.
3. **Dual-Model Verification (Judge Pattern)**:
   - Pass synthesized function arguments through an isolated, zero-privilege evaluator model before execution.
4. **Output Sanitization & Exfiltration Defenses**:
   - Restrict rendering of untrusted markdown images (`![]()`) or outbound hyperlinks that encode sensitive variables in query strings.

---

## ⚠️ Disclaimer
All research, payloads, and scenarios documented here are provided strictly for educational purposes, defensive benchmark evaluation, and red-teaming security readiness. All tests were executed against local testbeds, self-hosted environments, or with explicit authorization.

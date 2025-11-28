# Aula 05 - DAST com OWASP ZAP

## 🎯 Objetivo

Implementar testes dinâmicos de segurança (DAST) usando OWASP ZAP no pipeline.

## 📹 Vídeos desta Aula

| Vídeo | Tema | O que você vai fazer |
|-------|------|---------------------|
| 01 | Fundamentos DAST | Entender SAST vs DAST, configurar ZAP |
| 02 | Automatização | ZAP Baseline Scan no GitHub Actions |
| 03 | Fix & Verify | Analisar alertas e corrigir |

## 📁 Estrutura do Repositório

```
.
├── app.py
├── requirements.txt
├── Dockerfile.secure
├── .zap/
│   └── rules.tsv         # Regras personalizadas
├── .github/
│   └── workflows/        # (Criado durante a aula)
└── docs/
    ├── PASSO-A-PASSO.md
    └── CHEATSHEET.md
```

## ⚙️ Pré-requisitos

- [ ] Aula 04 concluída
- [ ] Aplicação rodando em staging (ECS)
- [ ] Secret `STAGING_URL` configurado

## 📚 Documentação

| Vídeo | Hands-on |
|-------|----------|
| 01 - Fundamentos DAST | [HANDS-ON-05-01.md](docs/HANDS-ON-05-01.md) |
| 02 - Automatização | [HANDS-ON-05-02.md](docs/HANDS-ON-05-02.md) |
| 03 - Fix & Verify | [HANDS-ON-05-03.md](docs/HANDS-ON-05-03.md) |

**Referência rápida**: [Cheatsheet](docs/CHEATSHEET.md)

---

**FIAP - Pós Tech DevSecOps**

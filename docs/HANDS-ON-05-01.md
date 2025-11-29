# 🎬 Vídeo 5.1 - Fundamentos de DAST

**Aula**: 5 - DAST com OWASP ZAP  
**Vídeo**: 5.1  
**Temas**: SAST vs DAST; OWASP ZAP; Tipos de scan; Ambiente de staging

---

## 🚀 Antes de Começar

> **Esta aula PRECISA da app rodando!** DAST testa aplicações em execução.

### Pré-requisitos

| Requisito | Como verificar |
|-----------|----------------|
| Docker instalado | `docker --version` |
| App rodando local | `docker run -p 5001:5000 ...` |
| OU App deployada | `curl http://<IP>:5001` |

### Primeira vez?
→ Faça clone do repositório `fiap-dclt-devsecops-aula05` e suba a app localmente primeiro

---

## 📚 Parte 1: SAST vs DAST

### Passo 1: Diferença Fundamental

```
SAST (Análise Estática)          DAST (Análise Dinâmica)
┌───────────────────────┐          ┌───────────────────────┐
│  Código Fonte        │          │  Aplicação Rodando   │
│       ↓              │          │         ↓            │
│    Análise           │          │   Ataque Simulado    │
│       ↓              │          │         ↓            │
│   Findings           │          │  Vulnerabilidades    │
└───────────────────────┘          └───────────────────────┘
```

| Aspecto | SAST | DAST |
|---------|------|------|
| **Quando** | Build time | Runtime |
| **O que analisa** | Código fonte | Aplicação rodando |
| **Como** | Leitura do código | Requisições HTTP |
| **Perspectiva** | White-box (vê tudo) | Black-box (atacante) |
| **Falsos positivos** | Mais | Menos |
| **Detecta** | Bugs no código | Vulnerabilidades exploráveis |

---

### Passo 2: Por que Precisamos de DAST?

**Problema**: SAST pode ter muitos falsos positivos

```
SAST encontra 100 issues
         ↓
   Quantos são reais?
    /           \
50 falsos     50 reais
positivos        ↓
          Quantos exploráveis?
                 ↓
            20 exploráveis
```

**DAST resolve isso**: Testa a aplicação real, encontra o que é realmente explorável!

---

### Passo 3: O que é OWASP ZAP?

**ZAP** = Zed Attack Proxy

**Características:**
- Projeto OWASP (gratuito)
- Scanner de vulnerabilidades web
- Proxy interceptador
- API para automação

**Tipos de Scan:**

| Tipo | Tempo | Descrição |
|------|-------|-----------|
| **Baseline** | 5-10 min | Passivo, não intrusivo |
| **Full** | 1h+ | Ativo, testa exploits |
| **API** | Variável | Para REST/GraphQL |

---

### Passo 4: O que ZAP Detecta?

**OWASP Top 10:**

| # | Vulnerabilidade | ZAP detecta? |
|---|-----------------|--------------|
| A01 | Broken Access Control | ✅ |
| A02 | Cryptographic Failures | ✅ |
| A03 | Injection | ✅ |
| A05 | Security Misconfiguration | ✅ |
| A06 | Vulnerable Components | ⚠️ |
| A07 | Authentication Failures | ✅ |

---

## 🍴 Parte 2: Configurar Repositório

### Passo 5: Fork e Clone

1. Acesse: `https://github.com/josenetoo/fiap-dclt-devsecops-aula05`
2. Clone:

**Linux/Mac:**
```bash
cd ~/fiap-devsecops
git clone https://github.com/josenetoo/fiap-dclt-devsecops-aula05.git
cd fiap-dclt-devsecops-aula05
ls -la
```

**Estrutura esperada:**
```
fiap-dclt-devsecops-aula05/
├── app.py
├── requirements.txt
├── Dockerfile.secure
├── .zap/
│   └── rules.tsv       ← Regras customizadas
└── docs/
```

---

## 🔍 Parte 3: Testar ZAP Localmente

### Passo 6: Subir Aplicação Local

**Linux/Mac:**
```bash
cd ~/fiap-devsecops/fiap-dclt-devsecops-aula05

# Build da imagem
docker build --platform linux/amd64 -t app:test -f Dockerfile.secure .

# Rodar container
docker run -d -p 5001:5000 --name app-test app:test

# Verificar
curl http://localhost:5001
```

**Windows (PowerShell):**
```powershell
cd ~\projetos\fiap-dclt-devsecops-aula05

# Build da imagem
docker build --platform linux/amd64 -t app:test -f Dockerfile.secure .

# Rodar container
docker run -d -p 5001:5000 --name app-test app:test

# Verificar
Invoke-WebRequest http://localhost:5001
```

---

### Passo 7: Executar ZAP Baseline Scan

**Linux/Mac:**
```bash
# ZAP Baseline Scan (via Docker)
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
  -t http://host.docker.internal:5001

# Nota: host.docker.internal permite acessar o host de dentro do container
```

**Windows (PowerShell):**
```powershell
# ZAP Baseline Scan (via Docker)
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py `
  -t http://host.docker.internal:5001
```

---

### Passo 8: Analisar Output

**Resultado esperado:**

```
WARN-NEW: Cookie No HttpOnly Flag [10010] x 2 
	http://host.docker.internal:5001/
WARN-NEW: Cookie Without Secure Flag [10011] x 2 
	http://host.docker.internal:5001/
WARN-NEW: X-Content-Type-Options Header Missing [10021] x 1 
	http://host.docker.internal:5001/
WARN-NEW: Content Security Policy (CSP) Header Not Set [10038] x 1 
	http://host.docker.internal:5001/

FAIL-NEW: 0	FAIL-INPROG: 0	WARN-NEW: 4	WARN-INPROG: 0	INFO: 0	PASS: 42
```

**Legenda:**
- **FAIL**: Vulnerabilidade crítica
- **WARN**: Alerta (boas práticas)
- **INFO**: Informativo
- **PASS**: Passou no teste

---

### Passo 9: Limpar Ambiente

**Linux/Mac:**
```bash
docker stop app-test
docker rm app-test
```

---

## ☁️ Parte 4: Preparar Ambiente de Staging

### Passo 10: Por que Staging?

```
DEV ──→ STAGING ──→ PROD
            ↑          ✗
           DAST    (NUNCA!)
```

> ⚠️ **NUNCA** execute DAST em produção! Pode causar:
> - Lentidão
> - Dados corrompidos
> - Alertas falsos de segurança

---

### Passo 11: Configurar URL de Staging

No GitHub, adicione secret:

1. **Settings** > **Secrets** > **Actions**
2. **New repository secret**
3. Name: `STAGING_URL`
4. Value: `http://<IP-DO-ECS>:5000`

> 💡 Use o Public IP da task ECS que você criou na Aula 01

---

## 🔧 Troubleshooting

| Erro | Causa | Solução |
|------|-------|---------|
| `host.docker.internal` não funciona | Linux sem suporte | Usar `--network host` |
| Timeout | App não está rodando | Verificar container |
| ZAP não encontra app | URL errada | Verificar porta e host |

---

**FIM DO VÍDEO 5.1** ✅

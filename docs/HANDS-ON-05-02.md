# 🎬 Vídeo 5.2 - Automatização com ZAP

**Aula**: 5 - DAST com OWASP ZAP  
**Vídeo**: 5.2  
**Temas**: ZAP no pipeline; Rules file; Relatórios; GitHub Actions

---

## 📚 Parte 1: ZAP no Pipeline

### Passo 1: Fluxo de DAST no CI/CD

```
Push → Build → Deploy Staging → ZAP Scan
                                    ↓
                              Alertas?
                             /        \
                      Críticos      Warnings
                          ↓            ↓
                       FALHA      Relatório → Deploy Prod
```

**Pontos importantes:**
- DAST precisa da aplicação **rodando**
- Executar **após** deploy em staging
- Pode ser agendado (não a cada push)

---

### Passo 2: Estratégias de Execução

| Estratégia | Quando | Vantagem |
|------------|--------|----------|
| **A cada PR** | Pull Request | Feedback rápido |
| **Agendado** | Noite/fim de semana | Não bloqueia |
| **Manual** | workflow_dispatch | Sob demanda |

> 💡 Recomendação: Agendado + Manual para começar

---

## 📝 Parte 2: Arquivo de Regras

### Passo 3: O que é rules.tsv?

**Rules file** = Configuração de como tratar cada alerta

**Formato:** `ID<TAB>ACTION<TAB>DESCRIPTION`

**Actions disponíveis:**

| Action | Comportamento |
|--------|---------------|
| `IGNORE` | Ignora o alerta |
| `WARN` | Alerta mas não falha |
| `FAIL` | Falha o pipeline |

---

### Passo 4: Criar Rules File

**Linux/Mac:**
```bash
cd ~/fiap-devsecops/fiap-dclt-devsecops-aula05

# Criar diretório
mkdir -p .zap

# Criar arquivo de regras
cat > .zap/rules.tsv << 'EOF'
# ZAP Rules Configuration
# Formato: ID	ACTION	DESCRIPTION
10010	WARN	Cookie No HttpOnly Flag
10011	WARN	Cookie Without Secure Flag
10015	WARN	Incomplete or No Cache-control Header
10021	WARN	X-Content-Type-Options Header Missing
10038	WARN	Content Security Policy (CSP) Header Not Set
10098	WARN	Cross-Domain Misconfiguration
90022	WARN	Application Error Disclosure
EOF
```

**Windows (PowerShell):**
```powershell
cd ~\projetos\fiap-dclt-devsecops-aula05

# Criar diretório
New-Item -ItemType Directory -Force -Path .zap

# Criar arquivo de regras
@"
# ZAP Rules Configuration
10010	WARN	Cookie No HttpOnly Flag
10011	WARN	Cookie Without Secure Flag
10015	WARN	Incomplete or No Cache-control Header
10021	WARN	X-Content-Type-Options Header Missing
10038	WARN	Content Security Policy (CSP) Header Not Set
10098	WARN	Cross-Domain Misconfiguration
90022	WARN	Application Error Disclosure
"@ | Out-File -FilePath .zap/rules.tsv -Encoding UTF8
```

---

## 🔄 Parte 3: Workflow DAST

### Passo 5: Criar Workflow

**Linux/Mac:**
```bash
cd ~/fiap-devsecops/fiap-dclt-devsecops-aula05
mkdir -p .github/workflows

cat > .github/workflows/dast.yml << 'EOF'
# ============================================
# WORKFLOW: DAST com OWASP ZAP
# ============================================
name: 🔍 DAST Scan

on:
  # Execução manual
  workflow_dispatch:
  
  # Agendado: Segunda às 2h
  schedule:
    - cron: '0 2 * * 1'

# Permissões para criar issues automaticamente
permissions:
  issues: write
  contents: read

jobs:
  # ============================================
  # JOB: ZAP Baseline Scan
  # ============================================
  zap-baseline:
    name: 🕷️ OWASP ZAP
    runs-on: ubuntu-latest
    
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4

      # Action oficial do ZAP
      # Nota: bug conhecido no upload de artifact interno, mas scan e issue funcionam
      - name: 🕷️ ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.14.0
        continue-on-error: true
        with:
          target: ${{ secrets.STAGING_URL }}
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
          issue_title: '🔴 ZAP DAST - Vulnerabilidades Encontradas'
          fail_action: false
EOF
```

---

### Passo 6: Entender o Workflow

```
Trigger (Manual/Agendado)
         ↓
      Checkout
         ↓
      ZAP Scan ──→ Vulnerabilidades? ──→ Cria Issue
```

**Parâmetros importantes:**

| Parâmetro | Descrição |
|-----------|-----------|
| `target` | URL da aplicação (secret) |
| `rules_file_name` | Arquivo de regras |
| `cmd_options: '-a'` | Ajax spider habilitado |
| `issue_title` | Título da issue criada automaticamente |
| `fail_action: false` | Não falha o pipeline com warnings |
| `continue-on-error: true` | Ignora bug de artifact da action |

**Permissões necessárias:**

```yaml
permissions:
  issues: write    # Criar issues automaticamente
  contents: read   # Ler arquivos do repositório
```

> 💡 Quando o ZAP encontra vulnerabilidades, cria automaticamente uma **Issue** no repositório com os detalhes!

> ⚠️ **Nota**: A action oficial tem um bug conhecido no upload de artifacts interno. O `continue-on-error: true` permite que o workflow seja marcado como sucesso mesmo com esse warning.

---

### Passo 7: Commit e Push

**Linux/Mac:**
```bash
git add .zap/rules.tsv .github/workflows/dast.yml
git commit -m "feat: adicionar DAST com ZAP"
git push origin main
```

**Windows (PowerShell):**
```powershell
git add .zap/rules.tsv .github/workflows/dast.yml
git commit -m "feat: adicionar DAST com ZAP"
git push origin main
```

---

## 🚀 Parte 4: Executar e Analisar

### Passo 8: Executar Manualmente

1. GitHub > **Actions**
2. Clique em **DAST Scan**
3. Clique em **Run workflow**
4. Aguarde execução (~5-10 min)

---

### Passo 9: Verificar Issue Criada

1. GitHub > **Issues**
2. Procure por: **🔴 ZAP DAST - Vulnerabilidades Encontradas**
3. A issue contém:
   - Lista de alertas encontrados
   - Severidade de cada alerta
   - URLs afetadas
   - Sugestões de correção

---

### Passo 10: Analisar a Issue

A issue criada automaticamente terá este formato:

```
🔴 ZAP DAST - Vulnerabilidades Encontradas

## Alertas Encontrados

| Alerta | Risco | Contagem |
|--------|-------|----------|
| X-Content-Type-Options Missing | Low | 3 |
| CSP Header Not Set | Medium | 2 |
| Server Version Disclosure | Low | 3 |

## Detalhes
- **X-Content-Type-Options Missing**
  - URL: http://...
  - Solução: Adicionar header X-Content-Type-Options: nosniff
```

---

## 🔧 Troubleshooting

| Erro | Causa | Solução |
|------|-------|---------|
| `Target unreachable` | URL errada ou app down | Verificar STAGING_URL |
| Timeout | App lenta | Aumentar timeout |
| Muitos falsos positivos | Rules não configuradas | Ajustar rules.tsv |
| Warning de artifact | Bug da action oficial | Ignorar (continue-on-error) |
| Issue não criada | Falta permissão | Verificar `issues: write` |

---

**FIM DO VÍDEO 5.2** ✅

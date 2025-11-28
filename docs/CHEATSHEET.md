# Aula 05 - DAST Cheatsheet

## ZAP Docker

```bash
# Baseline scan
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
  -t http://target:5001

# Full scan
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py \
  -t http://target:5001

# API scan
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py \
  -t http://target:5001/openapi.json -f openapi
```

## Rules.tsv

| Action | Descrição |
|--------|-----------|
| IGNORE | Ignorar alerta |
| WARN | Alertar mas não falhar |
| FAIL | Falhar pipeline |

## Alertas Comuns

| ID | Alerta |
|----|--------|
| 10010 | Cookie No HttpOnly |
| 10011 | Cookie Without Secure |
| 10021 | X-Content-Type-Options Missing |
| 10038 | CSP Header Not Set |

## Security Headers (Flask)

```python
@app.after_request
def add_headers(response):
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    return response
```

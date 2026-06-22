# appsec-workflows-v2

Vibe coding projeleri için **tek push'ta** SAST / SCA / Container / DAST + secret taraması yapıp
`GREEN / YELLOW / RED` kararı üreten reusable GitHub Actions pipeline'ı.

> Bu repo, [`ahmtcnn/appsec-workflows`](https://github.com/ahmtcnn/appsec-workflows) (v1) tabanlı
> **v2** geliştirme hattıdır. v1 olduğu gibi sabit kalır; yeni geliştirmeler burada yapılır.

## Ne yapar

Her `push`'ta otomatik koşan job'lar:

| # | Job | Araç | Kontrol |
|---|-----|------|---------|
| 0 | Preflight | `.appsec.yml` + file detection | Proje profili (dil, port, Dockerfile, components) |
| 1 | Secret Scan | Gitleaks | Hardcoded credential |
| 2 | SAST | Semgrep | Kod zafiyetleri (SQLi, XSS, cmd inj) |
| 3 | SCA | Trivy fs | Dependency CVE |
| 4 | Container Scan | Trivy image | Docker image CVE |
| 5 | DAST | ZAP + Nuclei | Çalışan uygulamaya aktif tarama |
| 6 | Verdict | — | GREEN / YELLOW / RED + commit status |

**Verdict (bankacılık sıfır tolerans):**
- 🔴 **RED** — secret veya CRITICAL bulgu → deploy yasak
- 🟡 **YELLOW** — HIGH bulgu → AppSec onayı gerekli
- 🟢 **GREEN** — temiz → deploy OK

## Kullanım

Projenin köküne `.github/workflows/security.yml` ekle:

```yaml
name: Security
on: { push: {} }
permissions: { contents: read, statuses: write, actions: read }
jobs:
  appsec:
    uses: mertcanzbey/appsec-workflows-v2/.github/workflows/security-reusable.yml@main
```

Opsiyonel olarak kök dizine `.appsec.yml` koy (daha akıllı pipeline):

```yaml
language: python      # python | nodejs | go | java
port: 5000
health_endpoint: /
dast: true
```

Proje standartları için [`standartlar.txt`](standartlar.txt) dosyasına bakın.

## v2 Yol Haritası

Geliştirmeler bu repoda yapılacak. (Detaylar eklenecek.)

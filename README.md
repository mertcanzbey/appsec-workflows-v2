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
| 5b | **CodeQL** 🆕 | CodeQL (in-pipeline) | SAST, `security-extended`, tam bu commit |
| 5c | **GHAS Native** 🆕 | Dependabot + Secret scanning | GitHub Advisory DB + doğrulanmış secret |
| 6 | Verdict | — | GREEN / YELLOW / RED + commit status |

### v2 yeniliği: GitHub Advanced Security entegrasyonu

GitHub'ın kendi taramaları (Dependabot, secret scanning) ve CodeQL artık verdict'e dahil.
CRITICAL/HIGH bulgu → RED/YELLOW. Ayrıca **fail-closed**: gerekli GHAS özelliği repo'da
kapalıysa "0 alert" güvenli sayılmaz → RED. Token kurulumu için
[docs/ENTERPRISE.md](docs/ENTERPRISE.md).

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
    secrets:
      ghas_token:           ${{ secrets.APPSEC_GHAS_TOKEN }}      # kişisel/test (PAT)
      ghas_app_id:          ${{ secrets.GHAS_APP_ID }}            # kurumsal (GitHub App)
      ghas_app_private_key: ${{ secrets.GHAS_APP_PRIVATE_KEY }}   # kurumsal (GitHub App)
```

> GHAS alert'lerini okumak için token gerekir (default `GITHUB_TOKEN` yetmez).
> Kurumsalda tek bir GitHub App tüm repoları karşılar — bkz [docs/ENTERPRISE.md](docs/ENTERPRISE.md).

Opsiyonel olarak kök dizine `.appsec.yml` koy (daha akıllı pipeline):

```yaml
language: python      # python | nodejs | go | java
port: 5000
health_endpoint: /
dast: true
```

Proje standartları için [`standartlar.txt`](standartlar.txt) dosyasına bakın.

## v2 Yol Haritası

- [x] GHAS native alert entegrasyonu (Dependabot + Secret scanning) + fail-closed compliance
- [x] CodeQL in-pipeline (`security-extended`, deterministik SARIF)
- [x] Kurumsal GitHub App ile token (org-level, per-repo token yok)
- [ ] (sonraki) SARIF birleştirme / GitHub Security tab entegrasyonu

# Kurumsal Kurulum — GHAS Token (GitHub App)

GHAS alert API'leri (Dependabot, Secret scanning) workflow'un default `GITHUB_TOKEN`'ı
ile okunamaz. Kurumsalda **tek bir GitHub App** tüm repoları karşılar — her repo için
ayrı PAT üretmeye gerek yoktur.

## Neden GitHub App (PAT değil)

| | PAT (per-repo) | GitHub App (tek) |
|---|---|---|
| 1000 repo için token | 1000 | **1** |
| Yeni repo eklenince | yeni token | **otomatik kapsanır** |
| Rotasyon | her birini tek tek | **tek private key** |
| Kime bağlı | bir kişiye | **App kimliğine** |
| Vibecoder ne yapar | token ekler | **hiçbir şey** |

## Adımlar (AppSec ekibi, bir kerelik)

1. **App oluştur:** Org → Settings → Developer settings → GitHub Apps → New GitHub App
   - Repository permissions (hepsi **Read-only**):
     - `Dependabot alerts`
     - `Secret scanning alerts`
     - `Administration`  (enablement/compliance kontrolü için)
     - `Metadata` (otomatik)
   - Webhook: kapalı (gerek yok)

2. **Private key üret:** App sayfasında "Generate a private key" → `.pem` iner.

3. **Org'a kur:** App → Install App → org → **All repositories**
   (yeni açılan repolar otomatik kapsanır).

4. **Org secret'ları tanımla:** Org → Settings → Secrets and variables → Actions:
   - `GHAS_APP_ID`          = App ID (App sayfasında)
   - `GHAS_APP_PRIVATE_KEY` = `.pem` dosyasının tam içeriği
   - Secret'ları hedef repolara (veya tümüne) görünür yap.

## Caller (her projede aynı)

```yaml
name: Security
on: { push: {} }
permissions: { contents: read, statuses: write, actions: read }
jobs:
  appsec:
    uses: <ORG>/appsec-workflows-v2/.github/workflows/security-reusable.yml@main
    secrets:
      ghas_app_id:          ${{ secrets.GHAS_APP_ID }}
      ghas_app_private_key: ${{ secrets.GHAS_APP_PRIVATE_KEY }}
```

Pipeline runtime'da `actions/create-github-app-token` ile o repoya özel, **1 saat ömürlü**
installation token üretir. Token önceliği: **App > PAT** (ikisi de varsa App kullanılır).

## Enforcement (RED'in gerçekten bloklaması için)

`verdict` RED → `AppSec / Verdict` commit status `failure` olur. Bunun merge/deploy'u
**bloklaması** için org seviyesinde zorunlu kılınmalı:

- Org → Settings → Rulesets → yeni ruleset
- Branch: `main` (ve gerekiyorsa diğer korunan dallar)
- "Require status checks to pass" → `AppSec / Verdict` ekle
- Mümkünse caller workflow'unu da org ruleset ile zorunlu kıl (silinmesini engelle).

## Repo gereksinimleri (vibecoder tarafı)

`enforce_ghas=true` (varsayılan) ise her repoda açık olmalı:
- Dependabot alerts → ENABLED
- Secret scanning → ENABLED

Kapalıysa fail-closed: "0 alert" güvenli sayılmaz → **RED**.
Code scanning'i repo'da açmaya gerek yok; CodeQL pipeline içinde koşar.

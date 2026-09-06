# ECC kademeli kurulum planı (pilot proje için)

> Bu dosyayı pilot projedeki ajana ver: "Bu listeyi al, üzerinden gideceğiz." Ajan her aşamada
> yalnız o aşamanın komutlarını çalıştırır, ölçümü `ECC-LOG.md`'ye yazar, çıkış kriteri
> sağlanmadan ve kullanıcı "geç" demeden bir sonraki aşamaya geçmez.

## Değişmez kurallar (her aşamada)

1. **Harness başına tek kurulum yolu.** Bu planda yol = proje-yerel seçici kurulum
   (`--target claude-project`). Claude eklentisi (`ecc@ecc`) AYRICA kurulmaz; ikisi üst üste
   binerse skill/hook çiftlenir.
2. **Sürüm sabit:** ECC `v2.2.1` / commit `e04ea0b9`. Her aşamadan önce `--dry-run`, sonra
   `npx ecc-universal doctor`.
3. **Ölçüm olmadan aşama yok.** Her aşamada aynı üç sayı: oturum başı bağlam yükü (`/context`),
   dilim başına token, hook gecikmesi. Baz aşama 0'dır.
4. **Geri alma her zaman hazır:** `npx ecc-universal uninstall` + `git status` temiz.
5. **Hook'lar asla push/force/hook-atlama yapamaz.** Proje ao ile yönetiliyorsa ECC'nin
   orkestrasyon komutları (`orch-*`, `multi-*`, loop) KAPALI kalır; koordinasyon ao'nundur.

## Aşama 0 — Baz ölçüm (ECC yok)

- Yap: 3 küçük dilimi ECC'siz tamamla; `/context` başlangıç yükünü, dilim başına token'ı,
  tur süresini `ECC-LOG.md`'ye yaz. Proje ajan konfigürasyonunu `npx -y ecc-agentshield scan -p .`
  ile tara, notu kaydet.
- Çıkış: üç dilimin sayıları kayıtlı.

## Aşama 1 — `minimal` profil, hook yok

- Kur: `npx ecc-universal install --target claude-project --profile minimal --dry-run`, sonra
  aynı komut `--dry-run`'sız. Modüller: `rules-core` (yalnız TEK dil paketi + `common`),
  `agents-core`, `commands-core`, `platform-configs`, `workflow-quality`.
- Kullan: `/plan`, `/code-review`, `tdd-workflow`, `verification-loop`, `search-first`,
  `architecture-decision-records`.
- Ölç: bağlam yükü artışı, review bulgularının kaçının gerçekten işe yaradığı.
- Çıkış: 5 dilim; başlangıç bağlamı baza göre ≤ %15 artmış; review bulgularının ≥ yarısı
  uygulanmış. Sağlanmazsa: `rules-core`'u tek pakete indir ya da aşamayı bırak.

## Aşama 2 — `core` profil, hook runtime `minimal` → `standard`

- Kur: `npx ecc-universal install --target claude-project --profile core` ile `hooks-runtime`
  eklenir; `ECC_HOOK_PROFILE=minimal` ile başla, bir hafta sonra `standard`.
- Gelen: SessionStart/Stop özetleri, memory-persistence, PreToolUse güvenlik, kalite kapısı.
- Ölç: hook başına gecikme, yanlış bloklama sayısı.
- Çıkış: hiçbir hook > 2 sn; sıfır yanlış bloklama; `strict` yalnız güvenlik gereken projede.

## Aşama 3 — `developer` profil (iş akışı)

- Kur: `--profile developer` → ek modüller `framework-language` (projenin yığını),
  `database`, `orchestration`.
- Kullan: `prp-prd → prp-plan → prp-implement → prp-pr` hattı; `quality-gate`, `delivery-gate`,
  `git-workflow`, `github-ops`, `pr`, `docs-lookup`.
- ao'lu projede: `orchestration` modülünü kurma (`--modules` ile dışarıda bırak).
- Çıkış: bir özellik uçtan uca bu hatla inmiş; PR şablonu ve review çıktısı kullanılabilir.

## Aşama 4 — Öğrenme ve hafıza

- Kur: `--modules skill-unified-memory` ekle. Skill'ler: `continuous-learning`,
  `continuous-learning-v2`, `unified-memory`, `strategic-compact`, `context-budget`,
  `token-budget-advisor`, `rules-distill`, `hookify-rules`. Komutlar: `/learn`, `/learn-eval`,
  `/instinct-status`, `/instinct-export`, `/evolve`.
- Ölç: üretilen instinct sayısı, kurala terfi eden sayı, compaction sonrası kayıp.
- Çıkış: ≥ 3 instinct, en az biri kural olmuş; compaction sonrası görev listesi korunmuş.

## Aşama 5 — Güvenlik ve operasyon

- Kur: `--modules security,operator-workflows,optimization-workflows`. AgentShield:
  `npx -y ecc-agentshield watch` ve runtime PreToolUse politikası; `/cost-report`,
  `ecc-tools-cost-audit`, `platform-audit`.
- Kullan: `security-review`, `security-scan`, `gateguard`, `safety-guard`; yığına göre
  `*-security` skill'i.
- Çıkış: AgentShield notu A; bilerek ekilen bir ihlali runtime politikası bloklamış;
  maliyet raporu aşama 0 baz sayılarıyla karşılaştırılmış.

## Aşama 6 — İhtiyaca göre genişlik

- Yalnız gerekeni ekle, `full` profilini körlemesine kurma: `research-apis`,
  `business-content`, `social-distribution` (araştırma/içerik); `media-generation`;
  `devops-infra`; `machine-learning`; `document-processing`; `swift-apple`;
  `supply-chain-domain`; `prediction-market-skills`; `ito-compute`; `nasiko-control-plane`.
- Her modül için: kur → bir gerçek görevde kullan → ölç → kalsın/çıksın kararı `ECC-LOG.md`'de.

## Aşama 7 — ECC 2.0 denemesi (ayrı karar)

- `ecc2/` (Rust, alfa) çok-oturum kontrol düzlemi: `npx ecc-universal control-pane`.
- Amaç: oturum yönetimi, worktree orkestrasyonu, risk skoru ve review kontrollerini ao ile
  karşılaştırmak. Üretim projesinde değil, pilotta.
- Çıkış: yazılı karşılaştırma; ao'nun yetki/makbuz/bağımsız-review farkı korunuyorsa ECC 2.0
  yalnız görünürlük için kalır.

## Kayıt şablonu (`ECC-LOG.md`)

```
## Aşama N — <tarih>
kurulum komutu:
dry-run çıktısı özeti:
doctor: ok / sorun
ölçüm: bağlam başlangıç = , dilim/token = , hook gecikme = 
gözlem:
karar: kal / geri al / geç
```

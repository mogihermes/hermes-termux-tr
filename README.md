# Termux için Hermes Agent — Türkçe Rehber

> Bu depo, **[adybag14-cyber/termux-hermes](https://github.com/adybag14-cyber/termux-hermes)** projesindeki Termux Hermes kurulum/yapılandırma belgesinin Türkçe çevirisidir. Kaynak sürüm: `main`, Hermes `v0.20.6` / `v2026.8.27`. Teknik gerçekler için her zaman kaynak projeyi ve [resmî Hermes belgelerini](https://hermes-agent.nousresearch.com/docs) esas alın.
>
> **Son kaynak kontrolü:** 2026-09-21. Kaynak README değişiklikleri her gün denetlenir; anlamlı bir değişiklikte bu çeviri güncellenir. Bu, sürümün Termux APT deposunda yayımlandığı veya her yeni Hermes sürümüyle anında uyumlu olduğu anlamına gelmez.

## Hangi yolu seçmeliyim?

| İhtiyaç | Önerilen yol |
| --- | --- |
| Resmî dağıtım, en basit başlangıç | `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash` |
| Aşağıdaki paketli, yerel aarch64 kurulum | Topluluk APT deposu + `pkg install hermes-agent` |
| Hermes kaynak koduna katkı / özel dal | Resmî Termux sayfasındaki manuel kaynak kurulumu |

## Güvenlik ve platform uyarıları

- Bu rehberdeki APT deposu **NousResearch tarafından işletilmez, imzalanmaz, barındırılmaz veya denetlenmez**; `adybag14-cyber` topluluğu tarafından işletilir. Etkinleştirmek, o deponun imzalama anahtarına güvenmeyi gerektirir.
- `curl | bash` komutları uzaktan alınan betiği çalıştırır. Komutu uygulamadan önce URL ve kaynağı kontrol edin; resmî yol için yalnızca `hermes-agent.nousresearch.com` alan adını kullanın.
- Android/Termux, Hermes için **best-effort / Tier 2** platformdur. Android arka plandaki Termux işlerini uyutabilir; gateway sürekliliği yönetilen sunucu hizmeti gibi garanti edilmez.
- Docker tabanlı terminal yalıtımı Termux içinde yoktur. Yerel sesli yazıya döküm (`faster-whisper`) Android wheel'i olmadığı için test edilen yolda desteklenmez. Otomatik browser/Playwright kurulumu atlanır; Android'de browser araçlarını deneysel kabul edin.

Resmî Termux sınırlamaları ve alternatif kaynak kurulumu: [Android / Termux belgeleri](https://hermes-agent.nousresearch.com/docs/getting-started/termux).

> **Android'de sıfırdan başlıyorsan:** Termux kurulumu, izinler, resmî Hermes installer'ı ve sorun giderme için [Android Cihaza Hermes Agent Kurulumu — Termux Türkçe Rehberi](https://github.com/mogihermes/android-cihaza-hermes-kurulumu) ile başla. Bu depo paketli/native APT yolu ve geliştirici ayrıntıları içindir.

## Amaç

Bu proje, telefonların C ve Rust paketlerini cihaz üzerinde derlemesi yerine, Hermes Agent Termux profili için gerekli yerel Python tekerlerini (wheel) derler. Böylece aarch64 Termux üzerinde kurulabilir ikili paketler kullanılır.

## Tek komutla kurulum ve kurtarma

Yerel Termux aarch64 kurulumu için — yarım kalmış güncellemeden kalan `.git/index.lock` dahil — şu komutu kullanın:

```bash
curl -fsSL --retry 6 --retry-all-errors https://raw.githubusercontent.com/adybag14-cyber/termux-hermes/main/install.sh | bash
```

Kurtarma betiği şunları yapar:

- İmzalı APT deposunu doğrular.
- Yalnızca paket simülasyonu sorun gösterirse Termux ayna uyumsuzluğunu düzeltir.
- En güncel, değiştirilemez `hermes-agent` paketini iki kaynaktan devam edebilen indirme ile alır.
- Eksik `dpkg` işlemlerini tamamlar.
- Çakışan ikinci bir Python kurulumu yerine mevcut resmî Termux Python 3.13'ü kabul eder.
- `~/.hermes` dizinini korur ve yapılandırmayı taşır.
- Maliyetli 32K–65K istekleri önlemek için OpenRouter çıktı sınırını 8.192 token yapar.
- Sonraki güncellemelerin `pkg upgrade` ile yapılacağını doğrular ve gateway'i yeniden başlatır.

İndirme önce değiştirilemez GitHub yayın varlığını, ardından Oracle APT kaynağını dener. Yerel kurulum öncesinde sabitlenmiş SHA-256 sağlama toplamını doğrular. `pkg install` yarıda kalmışsa kalan indirme `~/.cache/hermes-recovery/` altında tutulur; aynı komutu yeniden çalıştırmak indirmeye kaldığı yerden devam eder. Başarılı kurulum bu önbelleği kaldırır.

Farklı bir pozitif çıktı sınırı için komuttan önce `HERMES_RECOVERY_MAX_TOKENS` ortam değişkenini ayarlayın.

## Yerel `pkg install hermes-agent` paketi

Mevcut Debian paket sürümü `0.20.6+termux2`'dir. Python bağımlılığı, 3.13 serisindeki kurulu Termux `python` paketini kabul eder; yoksa imzalı, yan yana `python3.13` paketi kullanılır. Bu yaklaşım, Termux Python 3.14'e geçtikten sonra doğrulanmış CPython 3.13 ABI'sini korurken `pydoc3.13` sahiplik çakışmasını önler.

Depo, Hermes Agent için eksiksiz bir Termux `.deb` paketi de üretir. Bu ince bir Python wheel'i değildir: tam Hermes çalışma zamanı/kaynak varlıklarını ve doğrulanmış CPython 3.13 sanal ortamını içerir. Paket APT tarafından yönetilir; bu nedenle `hermes update`, paket sahipliğindeki dosyaları değiştirmez ve bunun yerine `pkg upgrade hermes-agent` komutunu bildirir.

İmzalı üçüncü taraf deposu bir kez şöyle etkinleştirilir:

```bash
curl -fsSL https://raw.githubusercontent.com/adybag14-cyber/termux-python/main/scripts/setup_apt_repo.sh | bash
```

Ardından Hermes ve yerel yardımcı paketler normal Termux paketleri olarak kurulabilir:

```bash
pkg install hermes-agent
pkg install wrangler
pkg install python3.13
```

Depo imza parmak izi:

```text
EAD24A2124EFA7393A78B7B14699F966313F7A6B
```

`build-hermes-package.yml`, sabitlenmiş resmî Termux Docker kullanıcı alanında çalışan yerel ARM GitHub runner'ında paketi kurar, çalışma zamanı içe aktarımlarını doğrular, `.deb` dosyasını oluşturur ve değiştirilemez GitHub yayını yayımlanmadan önce ikinci, temiz bir Termux kapsayıcısında bu paketi yeniden kurar. `apt-hermes-smoke.yml` ise canlı imzalı APT deposundan kurulumu ayrıca sınar.

Genel APT deposu, katkıcı tarafından işletilen bir dağıtım/doğrulama yoludur. Canonical Hermes Termux ikili varlıklarının nihayetinde NousResearch sahipliğindeki CI tarafından üretilip yayımlanması gereksiniminin yerini tutmaz.

## Sabit hedef

- Hermes kaynağı: `NousResearch/hermes-agent@5fc308a70719a83cccdbba4c0e39c23f5a8239d5` (`v2026.8.27`, Hermes 0.20.6)
- Denetlenen çalışma zamanı: Android 15/API 35 üzerinde resmî Termux uygulaması GitHub derlemesi `v0.118.3`
- ARM derleme ortamı: `sha256:3aed9c7fbcf9195a9919deaad418da006232864a779fb4f322d68a34887a2e15` ile sabitlenmiş resmî `termux/termux-docker` imajı
- Python: değiştirilemez `termux-aarch64-20260824.43.1` yayınından `3.13.15`
- Mimari: `aarch64`
- Wheel platformu: `android_24_arm64_v8a`
- Bağımlılık profili: Hermes `termux`

Her kaynak dağıtımının URL'si, sürümü ve SHA-256 değeri [`manifest/wheels.json`](https://github.com/adybag14-cyber/termux-hermes/blob/main/manifest/wheels.json) içinde tutulur. Mevcut yayın asla yerinde değiştirilmez; iş akışı, etiketi zaten olan bir yayını reddeder.

## Geliştirici ayrıntıları: yerel wheel kümesi

Mevcut Termux kilidi, Python 3.13 altında tam 74 paketi çözer. On paket Android'e özgü wheel gerektirir; kalanları ikili-only doğrulama kurulumu esnasında uyumlu ikili veya evrensel wheel'lerle sağlanır:

| Paket | Sürüm | Arka uç | Kilitlenmiş kaynak SHA-256 |
| --- | ---: | --- | --- |
| cffi | 2.0.0 | setuptools/C | `44d1b5909021139fe36001ae048dbdde8214afa20200eda0f64c068cac5d5529` |
| cryptography | 50.0.0 | maturin/Rust+CFFI | `eeac2acb5a20ed25e0ad6d1df9891a520b78b404266b6d11778f25d5d691a6c9` |
| jiter | 0.13.0 | maturin/Rust | `f2839f9c2c7e2dffc1bc5929a510e14ce0a946be9365fd1219e7ef342dae14f4` |
| MarkupSafe | 3.0.3 | setuptools/C | `722695808f4b6457b320fdc131280796bdceb04ab50fe1795cd540799ebe1698` |
| Pillow | 12.3.0 | setuptools/C | `3b8182a766685eaa002637e28b4ec8d6b18819a0c71f579bf0dbaa5830297cce` |
| psutil | 7.2.2 | setuptools/C + Android yaması | `0746f5f8d406af344fd547f1c8daa5f5c33dbc293bb8d6a16d80b4bb88f59372` |
| pydantic-core | 2.46.4 | maturin/Rust | `62f875393d7f270851f20523dd2e29f082bcc82292d66db2b64ea71f64b6e1c1` |
| PyYAML | 6.0.3 | setuptools/Cython/libyaml | `d76623373421df22fb4cf8817020cbb7ef15c725b9d5e45f17e189bfc384190f` |
| rpds-py | 0.30.0 | maturin/Rust | `dd8ff7cf90014af0c0f787eea34794ebf6415242ee1d6fa91eaba725cc441e84` |
| ruamel.yaml.clib | 0.2.15 | setuptools/Cython | `46e4cc8c43ef6a94885f72512094e482114a8a706d3c555a34ed4b0d20200600` |

Önceki 91 paketlik Android emülatör keşif kaydı, tarihsel kanıt olarak `audit/emulator-audit.json` içinde saklanır. Güncel sürüm için yeniden yayımlanmaz. Güncel, tam 74 paketlik çözücü çıktısı `audit/resolved.txt` dosyasındadır; doğrudan gereksinimler ve kilit kısıtları yanında tutulur.

## Derleme tasarımı

`Build immutable arm64 wheelhouse` iş akışı, yerel arm64 Ubuntu runner'ında çalışır ve sabit bir özetle belirtilmiş resmî aarch64 Termux Docker imajını kullanır. Paket keşfi ile son kurulum davranışı resmî Termux `v0.118.3` Android uygulamasında ayrı ayrı doğrulanmıştır.

Derleyici:

1. Sabitlenmiş Python `.deb` dosyasını ve her PyPI kaynak dağıtımını açmadan önce doğrular.
2. Kaynak arşivlerde yol dışına çıkma, sembolik bağ ve aygıt üyelerini reddeder.
3. Sabit setuptools, Cython, pybind11 ve maturin sürümleriyle seri derleme yapar.
4. `psutil` Android platform algısını yamalar.
5. PEP 738 Android wheel etiketlerini üretir veya normalleştirir.
6. Normalleştirme gerekirse `WHEEL` ve `RECORD` dosyalarını doğru biçimde yeniden yazar.
7. Paket/sürüm/etiket, ZIP bütünlüğü ve yerel uzantıların varlığını doğrular.
8. Tüm 74 paketlik grafiği temiz venv içinde `--only-binary :all:` ile kurar.
9. Her yerel paketi içe aktarır ve `uv pip check` çalıştırır.
10. Kesin Termux sistem paketi sürümlerini kaydeder.
11. Yeni ve değiştirilemez bir yayın etiketi altında wheel'leri, `index.json`, `system-packages.txt` ve `SHA256SUMS` dosyalarını yayımlar.

## Yerel aarch64 Termux'ta çalıştırma

```bash
git clone https://github.com/adybag14-cyber/termux-hermes.git
cd termux-hermes
bash scripts/termux_build.sh "$PWD"
```

Tam wheelhouse `~/termux-hermes-build/wheelhouse/` altında oluşturulur. Bu işlem kasıtlı olarak kaynak-yoğundur; normal kullanıcıların yayımlanmış varlıkları kullanması gerekir.

Otomatik arm64 yayın derlemesi, değiştirilemez özetle sabitlenmiş resmî Termux Docker imajını kullanır. Bu imaj, Termux ortamını cihaz dışında çalıştırmak için vardır; Android hedefli Python araç zincirinin gerektirdiği Termux bootstrap'i ve AOSP/Bionic çalışma zamanı bileşenlerini içerir. İş akışı `scripts/termux_build.sh /workspace /build docker` komutunu çağırır; açık kip, Docker doğrulamasını kapsayıcı giriş noktasınca temizlenen ortam değişkenlerinden bağımsız tutar.

## Kilidi yenileme

Paket sürümlerini, tam bir Hermes commit'ine göre Termux gereksinimlerini ve çözücü girdilerini yeniden üretmeden değiştirmeyin. Yenilemede aşağıdakiler birlikte güncellenmelidir:

- `audit/direct.in`
- `audit/lock-constraints.txt`
- `audit/resolved.txt`
- `manifest/wheels.json`
- kaynak README'deki wheel tablosu

`audit/emulator-audit.json` ile eski `audit/build-metadata.json`, onları üreten yayının tarihsel kanıtıdır; yeni kilidin kanıtıymış gibi yeniden etiketlenmemelidir. Güncel uyumluluk metadatası yerel Termux'ta şöyle üretilebilir:

```bash
python3.13 scripts/audit_pypi.py \
  --resolved audit/resolved.txt \
  --output audit/refreshed-emulator-audit.json
```

Yeni wheelhouse, yeni bir yayın etiketi kullanmalıdır. Var olan yayın varlıkları asla değiştirilmemelidir.

## Kaynaklar ve lisans

- Asıl belge: [adybag14-cyber/termux-hermes — README](https://github.com/adybag14-cyber/termux-hermes/blob/main/README.md)
- Paketleme kaynağı: [adybag14-cyber/termux-hermes](https://github.com/adybag14-cyber/termux-hermes)
- Hermes Agent belgeleri: [hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs)
- Hermes Agent kaynak kodu: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)

Bu çeviri, kaynak projenin MIT lisansı uyarınca yayımlanır. Özgün telif bildirimi ve lisans metni [`LICENSE`](LICENSE) dosyasındadır.

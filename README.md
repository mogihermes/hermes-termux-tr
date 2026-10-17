# Termux için Hermes Agent — Türkçe Rehber

> Bu depo, **[adybag14-cyber/termux-hermes](https://github.com/adybag14-cyber/termux-hermes)** projesindeki Termux Hermes paketleme belgesinin Türkçe çevirisidir. Kaynak sürüm: `main`, Hermes `v0.22.2`. Teknik gerçekler için her zaman kaynak projeyi ve [resmî Hermes belgelerini](https://hermes-agent.nousresearch.com/docs) esas alın.
>
> **Son kaynak kontrolü:** 2026-10-17. Kaynak README değişiklikleri her gün denetlenir; anlamlı bir değişiklikte bu çeviri güncellenir. Bu, sürümün Termux APT deposunda yayımlandığı veya her yeni Hermes sürümüyle anında uyumlu olduğu anlamına gelmez.

## Güvenlik ve platform uyarıları

- Bu rehberdeki APT deposu **NousResearch tarafından işletilmez, imzalanmaz, barındırılmaz veya denetlenmez**; `adybag14-cyber` topluluğu tarafından işletilir. Etkinleştirmek, o deponun imzalama anahtarına güvenmeyi gerektirir.
- `curl | bash` komutları uzaktan alınan betiği çalıştırır. Komutu uygulamadan önce URL ve kaynağı kontrol edin; resmî yol için yalnızca `hermes-agent.nousresearch.com` alan adını kullanın.
- Android/Termux, Hermes için **best-effort / Tier 2** platformdur. Android arka plandaki Termux işlerini uyutabilir; gateway sürekliliği yönetilen sunucu hizmeti gibi garanti edilmez.
- Docker tabanlı terminal yalıtımı Termux içinde yoktur. Yerel sesli yazıya döküm (`faster-whisper`) Android wheel'i olmadığı için test edilen yolda desteklenmez. Otomatik browser/Playwright kurulumu atlanır; Android'de browser araçlarını deneysel kabul edin.

Resmî Termux sınırlamaları ve alternatif kaynak kurulumu: [Android / Termux belgeleri](https://hermes-agent.nousresearch.com/docs/getting-started/termux).

> **Android'de sıfırdan başlıyorsan:** Termux kurulumu, izinler, resmî Hermes installer'ı ve sorun giderme için [Android Cihaza Hermes Agent Kurulumu — Termux Türkçe Rehberi](https://github.com/mogihermes/android-cihaza-hermes-kurulumu) ile başla. Bu depo paketli/native APT yolu ve geliştirici ayrıntıları içindir.

## Termux için Hermes

Bu depo, yerel **Termux aarch64** için [Hermes Agent](https://github.com/NousResearch/hermes-agent) paketler. Telefonlar hazırlanmış bir Debian paketi kurar; C ve Rust bağımlılıkları yerel ARM GitHub Actions runner'larında derlenir ve denetlenir.

## Kurulum veya kurtarma

Termux'ta şunu çalıştırın:

```bash
curl -fsSL --retry 6 --retry-all-errors https://raw.githubusercontent.com/adybag14-cyber/termux-hermes/main/install.sh | bash
```

Kurucu, imzalı paket deposunu denetler, gözden geçirilmiş değiştirilemez paketi devam edebilen aktarımlarla indirir, SHA-256 değerini doğrular ve yarım kalmış bir kurulumu onarır. `~/.hermes` dizinini korur, mevcut yapılandırmayı taşır, yerleşik 8.192 token çıktı sınırını korur ve gateway'i yeniden başlatır. Farklı bir pozitif sınır seçmek için `HERMES_RECOVERY_MAX_TOKENS` değişkenini ayarlayın.

Güncel paket, kendi Python ve yerel kütüphanelerini `$PREFIX/lib/hermes-agent` altında içerir. Mevcut bir Termux Python 3.13 veya 3.14 kurulumu ile birlikte çalışabilir. Hermes'i çalıştırmak için sistem Python'unu değiştirmek gerekmez.

İmzalı depoyu zaten kullanan bir kurulum için:

```bash
pkg update
pkg upgrade hermes-agent
```

Hermes APT'nin sahipliğinde kalır. `hermes update`, güncellemelerin `pkg upgrade hermes-agent` ile yapıldığını açıklar ve 2 durum koduyla çıkar.

## Yayın hedefi

Gözden geçirilmiş yayın girdileri [manifest/release.json](manifest/release.json) içindedir.

| Öğe | Hedef |
| --- | --- |
| Hermes | [v0.22.2](https://github.com/NousResearch/hermes-agent/releases/tag/v0.22.2) |
| Kesin kaynak | `d923a5eb487a17c0a68801bba9ef0f56c81040a8` |
| Debian paketi | `hermes-agent_0.22.2-1_aarch64.deb` |
| Mimari | aarch64 / arm64-v8a |
| Python | Paketlenmiş Termux CPython 3.14.6 |
| Wheel ABI ve platform | `cp314`, `android_24_arm64_v8a` |
| Özellik profili | Hermes `termux`: Telegram, cron, MCP ve ACP |
| Derleme kullanıcı alanı | `termux/termux-docker@sha256:c14ffb703abf44f2d3bf7fdffdcc50df7d1454d09c705be190e0342a0b47f303` |

v0.22.2 paketi upstream PM çalışma zamanını kullanır ve Node terminal arayüzünü derler. Python, Node, npm, uv, FFmpeg, ripgrep, çalışma zamanı paylaşımlı kütüphaneleri, uygulama kaynağını ve yerel bağımlılık ortamını içerir. Hermes 0.20.6'daki Python 3.13 wheelhouse'u bu yayın için kullanılmaz.

Paket düzeni şöyledir:

- `lib/hermes-agent/app`: uygulama kaynağı ve yayın damgası.
- `lib/hermes-agent/venv`: uygulama bağımlılık ortamı.
- `lib/hermes-agent/pm-runtime`: yalıtılmış paket yöneticisi ortamı.
- `lib/hermes-agent/tools`: paketlenmiş araç ikilileri.
- `lib/hermes-agent/runtime-libs`: doğrulanmış yerel kütüphane kapanışı.
- `lib/hermes-agent/bin`: üretilmiş başlatıcılar; Termux'un `bin` dizini bunlara bağlanır.

Yayın damgası upstream etiketi ve commit'ini, `apt-termux` dağıtımını ve haricî güncelleme sahipliğini kaydeder. Ayrı paket revizyonu `-1`, upstream Termux oluşturucusunun Debian sürüm biçimini izler.

## İmzalı APT deposu

APT'yi doğrudan kullanmayı tercih ediyorsanız katkıcı tarafından işletilen depoyu bir kez etkinleştirin:

```bash
curl -fsSL --retry 6 --retry-all-errors https://raw.githubusercontent.com/adybag14-cyber/termux-python/main/scripts/setup_apt_repo.sh | bash
pkg install hermes-agent
```

Depo imza parmak izi:

```text
EAD24A2124EFA7393A78B7B14699F966313F7A6B
```

Mevcut yayıncı, doğrulanmış ve ön yayın olmayan GitHub yayın varlıklarını içe aktarır, paket metadatasını ve GitHub SHA-256 özetini denetler, ardından imzalı bir APT anlık görüntüsü yayımlar. Aday Actions varlıkları bu akışa girmez. Yayımlanmış sürümler ve varlıkları değiştirilemezdir.

Bu bağımsız bir katkıcı dağıtımıdır. Uygulama tanımlanan NousResearch sürümünden gelir; paketleme uyarlamaları ve doğrulama kanıtları burada korunur.

## Derleme ve yayın

[Build native Hermes Termux package](.github/workflows/build-hermes-package.yml), gözden geçirilmiş kesin kaynağı kullanır ve [patches](patches) içindeki denetlenmiş yamaları uygular. Derleme tam Termux özellik profilini korur ve güncel derleme araçlarını kurmadan önce sürekli güncellenen derleme kapsayıcısını yükseltir.

İş akışı:

1. Kararlı upstream etiketinin, kaydedilmiş kesin commit'e çözümlendiğini doğrular.
2. Kilitli ana makine araç zincirini upstream PM aracılığıyla sağlar.
3. Özetle doğrulanmış bionic araçları ve çalışma zamanı kütüphanelerini aşamalar. Upstream'in herkese açık, içerik-adresli aynası, Termux'un sürekli güncellenen havuzundan çıkarılan arşivlenmiş girdileri sağlar.
4. Termux profili için dondurulmuş upstream bağımlılık grafiğini dışa aktarır.
5. Yerel wheel'leri, sabitlenmiş aarch64 Termux kullanıcı alanında paketlenmiş hedef Python ile derler; ardından eksiksiz çevrimdışı bağımlılık kurulumunu ve yerel içe aktarma denetimlerini yapar.
6. Terminal arayüzünü derler ve standart PM tabanlı Debian paketini birleştirir.
7. Gerçek paketi ağ erişimi devre dışı bırakılmış yeni bir sabitlenmiş kapsayıcıya kurar; CLI, ACP, yerel uzantılar, PM bütünlüğü, Node/npm, ripgrep, FFmpeg ve terminal arayüzünün kurulum ekranını denetler.
8. Önceki paketin gerçek bir kurulumunu, mevcut Python 3.13 ve kullanıcı yapılandırmasıyla yükseltir; ardından korunan durumu ve başlatıcı sahipliğini denetler.
9. Paket ile bağımlılık kökenini kaydeder ve `hermes-agent-termux-aarch64` aday varlığını yükler.

Bu iş akışı yayımlamadan bir aday oluşturur. Elle şununla çalıştırın:

```bash
gh workflow run build-hermes-package.yml --ref <reviewed-build-branch>
```

Başarılı olduğunda paketleme ve yayın iş akışı değişikliklerini gözden geçirin ve birleştirin. Yeni varlık yayımlanana kadar genel kurucunun mevcut yayın sabitlemelerini koruyun. Adayın sağlama toplamını, paket sürümünü ve değiştirilemez yayın URL'sini gözden geçirilmiş bir yayın dalındaki `install.sh` dosyasına kaydedin. [Publish verified Hermes Termux candidate](.github/workflows/publish-hermes-package.yml) iş akışını, başarılı derlemenin çalıştırma kimliği ve yeni yayın etiketiyle o dalda başlatın:

```bash
gh workflow run publish-hermes-package.yml --ref <reviewed-release-branch> \
  -f build_run_id=<successful-native-run-id> \
  -f release_tag=<new-release-tag>
```

Yükseltme, bu kesin adayı yeniden derlemeden indirir. Başarılı çalıştırmayı, paket kimliğini, upstream sabitlemelerini, sağlama toplamlarını ve kurucu sabitlemelerini doğrular; mevcut paket yükseltmesini yeniden çalıştırır; ayrıca depodaki akışlı kurtarma kurucusunu gerçek, yarım kalmış bir HTTP aktarımıyla sınar. Ardından taslak bir yayın oluşturur, her yüklenen varlığın GitHub özetini ve boyutunu denetler, son olarak yayını yayımlar. Mevcut bir etiket veya yayın reddedilir. Kurtarma ayrıca eski Termux tabanının GPG/libgcrypt uyumsuzluğunu yeniden üretir ve kurucunun sistem Python'unu korurken bu bağımlılık kümesini onarmasını gerektirir.

Yayınlama başarılı olduğunda kurucu sabitlemelerini `main` dalına birleştirin. Yeni paket imzalı APT deposunda göründükten sonra canlı APT smoke denetimini bu commit üzerinde çalıştırın. Bu sıralama, genel kurucunun mevcut bir varlığa işaret etmesini sağlar.

Her paket yayını şunları içerir:

- `SHA256SUMS`: Debian paketinin ve ekli kanıtların sağlama toplamları.
- `BUILD-METADATA.json`: gerçek paket özeti, upstream ve paketleme commit'leri, derleme imajı, kaynak yama özetleri ve çalışma zamanı sözleşmesi incelemesi.
- `WHEELHOUSE-MANIFEST.json`: wheel özetleri ve tam önbellek girdi kimliği.
- `RESOLVED.txt`: kilitli sürümler, hedef işaretleyiciler ve kaynak başvuruları.
- `RELEASE-PINS.json`: gözden geçirilmiş yayın hedefi.

Ayrı [live APT smoke](.github/workflows/apt-hermes-smoke.yml), imzalı genel depo üzerinden kurulumu ve yarım kalmış indirme kurtarmasını doğrular. Yayın derleme doğrulaması ile canlı APT doğrulaması ayrı kapılardır.

## Hedefi güncelleme

Kesin kararlı etiketi ve commit'i `manifest/release.json` içinde birlikte güncelleyin. Derlemeden önce upstream çalışma zamanı, Python ABI, PM ve yerel bağımlılık değişikliklerini gözden geçirin. Derleme yamalarını bu kaynağa göre yenileyin ve inceleyin. Yayın sürümünü `pyproject.toml` içindeki upstream geliştirme değerinden çıkarsamayın; doğrulanmış kararlı etiket ve kurulum damgası yayını tanımlar.

Bir aday yerel kapılardan geçtikten sonra kurtarma paketi sürümünü, değiştirilemez varlık URL'sini ve SHA-256 değerini birlikte güncelleyin. Canlı smoke'un beklenen sürümü, kısmi indirme düzeneği ve önbellek adları yayınla birlikte değişmelidir. APT yayıncısı en yeni etkin paketi tutar; bu nedenle eski etkin havuz URL'leri kalıcı düzenekler değildir; tarihsel paketler değiştirilemez GitHub URL'lerini kullanmalıdır.

## Tarihsel derlemeler ve cihaz kanıtı

[Hermes 0.20.6 / CPython 3.13 build documentation](docs/legacy-cp313.md), `manifest/wheels.json` ve mevcut `audit/` dosyaları önceki yayın hattını açıklar. Eski wheelhouse iş akışı ve betikleri, bu hattı yeniden üretmek için kullanılabilir durumda kalır. Bunlar v0.22.2 için kanıt değildir.

Yeni yayının yerel kapsayıcı doğrulaması, tek başına belirli bir fiziksel telefonu sertifikalandırmaz. Cihaza özgü kanıt, toplandığında Android sürümünü, Termux uygulamasını ve sınanan paket sağlama toplamını belirtmelidir.

## Kaynaklar ve lisans

- Asıl belge: [adybag14-cyber/termux-hermes — README](https://github.com/adybag14-cyber/termux-hermes/blob/main/README.md)
- Paketleme kaynağı: [adybag14-cyber/termux-hermes](https://github.com/adybag14-cyber/termux-hermes)
- Hermes Agent belgeleri: [hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs)
- Hermes Agent kaynak kodu: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)

Bu çeviri, kaynak projenin MIT lisansı uyarınca yayımlanır. Özgün telif bildirimi ve lisans metni [`LICENSE`](LICENSE) dosyasındadır.

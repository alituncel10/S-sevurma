# ŞİŞE VURMA

Sapanla renkli sıvılarla dolu **şeffaf cam şişeleri** vurduğun çizgi film tarzı bir mobil/masaüstü oyun.
Phaser 3 + düz JavaScript, derleme adımı yok, görsellerin ve seslerin tamamı kodla üretilir (harici dosya yok).

## 1. Yerel olarak çalıştırma

Gereken: [Node.js](https://nodejs.org) 18 veya üstü (yalnızca küçük bir yerel sunucu için).

```bash
cd sise-vurma
npm start                # http://localhost:8080
```

* **Bilgisayarda:** tarayıcıda `http://localhost:8080` adresini aç, fareyle oyna.
* **Telefonda:** telefon ve bilgisayar aynı Wi-Fi ağında olsun. `npm start` çıktısındaki
  `Telefonda : http://192.168.x.x:8080` adresini telefonun Chrome'unda aç ve telefonu **yatay** tut.
* Node yoksa: `python3 -m http.server 8080` de olur.
* `index.html` dosyasına çift tıklamak çoğu tarayıcıda çalışır, ama yazı tipi gibi yerel dosya kısıtları
  nedeniyle bir sunucu üzerinden açmak daha güvenlidir.

## 2. Nasıl oynanır

* Ekranın herhangi bir yerine bas, **geriye (sola/aşağı) çek**, bırak. Top fırlar. Fare ve dokunmatik aynı çalışır.
* Her bölümde **6 şişe** vardır, her şişenin sıvısı farklı renktir. Üstteki **HEDEF** bandı hangi renkli şişeyi
  vuracağını gösterir. 6 hedef **sırayla** vurulunca bölüm biter.
* Her bölüm **2 top hakkıyla** başlar (ekranda `2/2`).
  * **Doğru şişe:** puan kazanılır (100 + üst üste vuruşta seri bonusu), şişe patlar, top hakkı harcanmaz.
    Aynı atış kırılan şişeden geçip sıradaki hedefi de vurursa seri devam eder.
  * **Yanlış şişe ya da ıska:** 1 top hakkı gider.
  * **Top hakkı 0 olunca** "Topların bitti!" penceresi açılır:
    **REKLAM İZLE — 3 EKSTRA TOP KAZAN** ya da **BÖLÜMÜ YENİDEN BAŞLAT**.
* Bölüm bitince **BÖLÜM TAMAMLANDI!** ekranı: kazanılan puan, kalan top, toplam puan, yıldızlar
  (hatasız = 3 yıldız) ve **SONRAKİ BÖLÜM** düğmesi. Kalan her top 50 puan ekler.
* Alt köşede: ana menü, bölümü yeniden başlat, ses aç/kapat.
* **Bölümler:** 1. bölüm tek sıra, sabit platform (kolay). Sonrakilerde basamaklı platformlar, ayrı adalar,
  yana/yukarı-aşağı hareket eden platformlar, 6 şişe şekli ve 8 renkten 6'sı gelir; zorluk arttıkça yörünge
  noktaları azalır, hareketli platformlar hızlanır. Bölüm düzenleri `src/core/levels.js` içindeki şablonlardan üretilir.
* **Nişan yardımı:** bırakılan çekiş hedef şişeyi çok az ıskalıyorsa küçük bir "mıknatıs" etkisiyle
  düzeltilir (yalnızca hedef şişe için; ilerleyen bölümlerde zayıflar).

## 3. Ödüllü reklam sistemi

Reklam mantığı `src/core/ads.js` içindedir, oyun sahnesi yalnızca `AdManager` ile konuşur:

| Sağlayıcı | Ne zaman | Nasıl |
|---|---|---|
| `MockProvider` (sahte test reklamı) | Tarayıcıda / geliştirmede (varsayılan) | Ekranda "TEST REKLAMI" penceresi, 5 sn geri sayım |
| `AdMobProvider` | Capacitor ile paketlenmiş **Android uygulamasında** | `@capacitor-community/admob` eklentisi, AdMob Rewarded Ads |

Kurallar (kodda uygulanır ve testlerle doğrulanır):

* Reklam **yalnızca REKLAM İZLE düğmesine basılınca** gösterilir (arka planda yalnızca yüklenir).
* Reklam **sonuna kadar izlenirse** (ödül olayı gelirse) 3 top verilir; **erken kapatılırsa** top verilmez.
* Yüklenemezse/gösterilemezse hata mesajı ve **TEKRAR DENE** düğmesi çıkar.
* Aynı anda ikinci reklam isteği reddedilir (çift ödül yok).

### Geliştirmede test senaryoları

`config/ads.config.js` içinde `allowQueryOverrides: true` iken adrese parametre ekleyerek zorlayabilirsin:

| Adres | Davranış |
|---|---|
| `?ad=loadfail` | Reklam yüklenemez (hata + TEKRAR DENE) |
| `?ad=showfail` | Yükleme tamam, gösterme hata verir |
| `?ad=early` | Reklam erken kapatılmış gibi davranır (ödül yok) |
| `?adms=400` | Sahte reklam 400 ms sürer (hızlı test) |

### Gerçek AdMob reklamını açma (Android)

**AdMob, düz web sayfasında reklam gösteremez**; oyunu Capacitor ile Android uygulamasına sarman gerekir.
Oyun kodu buna hazırdır: uygulama içinde `provider: 'auto'` AdMob'u kendiliğinden seçer.

1. [AdMob](https://admob.google.com)'da uygulama ve **Ödüllü** reklam birimi oluştur. İki kimlik alırsın:
   *uygulama kimliği* (`ca-app-pub-…~…`) ve *reklam birimi kimliği* (`ca-app-pub-…/…`).
2. Reklam birimi kimliğini **yalnızca** `config/ads.config.js` içine yaz (`admob.rewardedAdUnitId.android`).
   Kodun içinde hiçbir reklam kimliği yoktur. Dosyadaki `testRewardedAdUnitId`, Google'ın herkese açık
   *örnek test* birimidir ([resmî belge](https://developers.google.com/admob/android/test-ads)); gelir getirmez.
3. Android uygulamasını oluştur:
   ```bash
   npm i @capacitor/core @capacitor/android @capacitor-community/admob
   npm i -D @capacitor/cli
   # capacitor.config.json içindeki appId'yi kendi paket adınla değiştir (örn. com.sirketim.sisevurma)
   npm run prepare:www          # web dosyalarını www/ klasörüne kopyalar
   npx cap add android
   npx cap sync android
   ```
4. `android/app/src/main/AndroidManifest.xml` içinde `<application>` altına ekle (AdMob **uygulama** kimliği):
   ```xml
   <meta-data android:name="com.google.android.gms.ads.APPLICATION_ID" android:value="@string/admob_app_id" />
   ```
   ve `android/app/src/main/res/values/strings.xml` içine `<string name="admob_app_id">ca-app-pub-…~…</string>` ekle.
   Yatay kilit için `<activity>` etiketine `android:screenOrientation="sensorLandscape"` ekleyebilirsin.
5. `npx cap open android` → Android Studio'dan çalıştır/derle. İlk denemelerde `useTestAds: true` kalsın.
6. **Yayından önce:** `useTestAds: false`, `allowQueryOverrides: false` yap ve kendi gerçek reklam birimi kimliğini gir.

> Not: Bu ortamda gerçek bir Android cihaz ve AdMob hesabı olmadığı için yerel AdMob akışı yalnızca
> sahte köprüyle (testler A7–A17) doğrulandı; gerçek cihazda ilk çalıştırmada denemen gerekir.

## 4. Proje yapısı

```
index.html                 Giriş sayfası (yatay çevir uyarısı, sahte reklam penceresi)
config/ads.config.js       Reklam ayarları ve reklam birimi kimlikleri (tek yer)
lib/phaser.min.js          Phaser 3.90.0 (MIT)
assets/fonts/              Baloo 2 yazı tipi (OFL, Türkçe karakterli)
src/main.js                Ekran ölçüleri, Phaser kurulumu, yön değişimi
src/core/                  Saf mantık: state (puan/top/bölüm), levels, ads, audio, storage
src/game/                  physics.js (fizik + çözücü), aim.js (girdi), sling.js, fx.js (efektler)
src/gfx/                   Kodla çizilen dokular: cam şişe + sıvı, platform, arka plan, top
src/ui/                    HUD, düğmeler, pencereler, sahte reklam ekranı
src/scenes/                Boot, Menu, Game sahneleri
tests/                     core.test.js, physics.test.js, e2e/run.js (tarayıcı testi)
tools/                     serve.js (yerel sunucu), prepare-www.js (Android için kopyalama)
```

**Teknik notlar:** Mantıksal ekran yüksekliği 720, genişliği cihaz oranına göre 1152–1584 arasıdır
(Phaser `FIT`); yüksek çözünürlüklü ekranlarda tuval 2 kata kadar büyütülür. Fizik sabit 1/240 sn
adımla çalışır, bu yüzden hızlı top ince şişe boyunlarından geçmez ve sonuç ekran yenileme hızından
bağımsızdır. Şişeler statik gövdelerdir (devrilmez), çarpınca sallanır, sıvı yaylı biçimde çalkalanır.

## 5. Testler

```bash
npm run test:fast     # 32 mantık testi (puan, top hakları, bölümler, reklam akışı) ~1 sn
npm test              # + fizik testi: 12 bölümdeki 72 şişenin hepsi vurulabilir mi (1280 genişlik, ~1-2 dk)
npm run test:full     # aynısı 960/1280/1600 genişliklerde, 216 şişe (~4 dk)
npm i -D playwright && npx playwright install chromium
npm run test:e2e      # gerçek Chromium'da fare/dokunma ile 8 senaryo (~5 dk, yazılım GL yavaş)
```

`tests/e2e/run.js`, "son kontrol" listesindeki senaryoları oyunun gerçek arayüzünde çalıştırır
(K1 2 top · K2 yanlış atış · K3 "Topların bitti!" · K4 reklamla 3 top · K5 reklam tamamlanmazsa/yüklenmezse ·
K6 puan ve bölüm geçişi · K7 şeffaf cam + 6 sıvı rengi (piksel analizi) · K8 dokunmatik telefon).
Ekran görüntüleri `tests/e2e/screenshots/` klasörüne kaydedilir. Tek senaryo: `node tests/e2e/run.js --only=K7`.

## 6. Lisanslar

Phaser: MIT (`lib/PHASER-LICENSE.md`). Baloo 2 yazı tipi: SIL OFL 1.1 (`assets/fonts/OFL-Baloo2.txt`).

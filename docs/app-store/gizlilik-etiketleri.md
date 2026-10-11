# App Store gizlilik etiketleri (App Privacy) — RevenueCat ve Firebase Crashlytics

Son güncelleme: 2026-10-11 · Kaynak: uygulama kodu `Eyyogang/deepgram` `main` (214ba3c) · Issue: Eyyogang/deepgram#100

> Bu belge App Store Connect → Uygulama Gizliliği (App Privacy) bölümüne **eklenecek** veri türlerini önerir.
> Hesap, e-posta, altyazı sesi gibi mevcut beyanlar bu belgenin kapsamı dışındadır; onlara dokunma.
> Etiketler gizlilik politikasıyla (`/gizlilik/`, `/en/privacy/`, `/de/datenschutz/`) tutarlı olmalıdır.

## Koddan çıkan gerçekler

- **RevenueCat** (`ios/App/Sources/Store/RevenueCatTracker.swift`): yalnız gözlemci (`purchasesAreCompletedBy: .myApp`, StoreKit 2).
  Tamamlanan satın almalar `recordPurchase`, "Geri yükle" `syncPurchases` ile bildirilir. Giriş yapılınca
  `appUserID` = hesabın UUID'si (küçük harf; StoreKit `appAccountToken` ile aynı), çıkışta `logOut` ile anonim
  müşteriye dönülür. E-posta veya ad gönderilmez; özel öznitelik (`setAttributes`, `setEmail`) ve
  `collectDeviceIdentifiers` çağrısı yok. Pro hakkı ve kota RevenueCat'e bağlı değildir.
- **Firebase Crashlytics** (`ios/App/Sources/App/CrashReporting.swift`): yalnız `FirebaseCrashlytics` ürünü bağlı;
  `GoogleService-Info.plist` içinde `IS_ANALYTICS_ENABLED` ve `IS_ADS_ENABLED` = false. Debug derlemesinde toplama
  kapalı, Release'de açık. `setUserID` ile hesap UUID'si eklenir (çıkışta boş dizeye çekilir). Özel anahtar
  (`setCustomValue`), log (`log`) veya elle hata kaydı (`record`) yok.
- **IDFA / izleme**: uygulama kodunda `AdSupport`, `AppTrackingTransparency` veya `advertisingIdentifier` yok.

## Eklenecek veri türleri

| Kategori | Veri türü | Kaynak | Amaç (Purposes) | Kullanıcıya bağlı mı? (Linked to user?) | İzleme için mi? (Used for tracking?) |
|---|---|---|---|---|---|
| Purchases | **Purchase History** | RevenueCat (ve kendi sunucumuzun abonelik doğrulaması) | App Functionality, Analytics | **Evet** | **Hayır** |
| Identifiers | **User ID** | RevenueCat `appUserID` ve Crashlytics `setUserID` (hesap UUID'si) | App Functionality, Analytics | **Evet** | **Hayır** |
| Diagnostics | **Crash Data** | Firebase Crashlytics (yığın izi, çökme anındaki uygulama durumu) | App Functionality | **Evet** | **Hayır** |
| Diagnostics | **Other Diagnostic Data** | Firebase Crashlytics (cihaz modeli, iOS sürümü, RAM/disk durumu) | App Functionality | **Evet** | **Hayır** |

Gerekçeler:

- **Linked to user = Evet**: RevenueCat müşterisi ve Crashlytics raporları hesabın UUID'siyle eşlenir; UUID de
  sunucumuzda e-posta/Apple kimliğiyle bağlı hesaba karşılık gelir. Apple tanımına göre bu "bağlı" sayılır.
  Giriş yapmamış kullanıcıda kimlik anonimdir, ama etiket uygulama geneli için verilir.
- **Used for tracking = Hayır**: Veri reklam amaçlı başka şirketlerin verileriyle birleştirilmiyor, veri
  aracılarıyla paylaşılmıyor; IDFA okunmuyor, ATT izni istenmiyor.
- **Purchase History amaçları**: RevenueCat'in resmi rehberi App Functionality ve Analytics'i birlikte işaretlemeyi
  istiyor ([RevenueCat – Apple App Privacy](https://www.revenuecat.com/docs/platform-resources/apple-platform-resources/apple-app-privacy)).
  Purchase History kendi sunucumuz nedeniyle zaten beyan edilmişse yalnız **Analytics** amacını ekle.
- **Crash Data amacı**: Apple "App Functionality" tanımı "minimize app crashes" ifadesini kapsar. Analytics
  işaretlenmez; Google Analytics kapalıdır.

## Eklenmeyecekler

- **Device ID**: IDFA/IDFV toplanmıyor. Crashlytics kurulum kimlikleri uygulamaya özel rastgele kimliklerdir
  (bkz. doğrulanmalı listesi).
- **Usage Data / Product Interaction**: Google Analytics kapalı, breadcrumb log yok.
- **Contact Info**: RevenueCat'e ve Crashlytics'e e-posta veya ad gitmez.

## Doğrulanmalı

1. **Other Diagnostic Data**: Cihaz/OS bilgisi çökme raporunun parçası; ayrıca beyan gerekip gerekmediği
   yoruma açık. Önerimiz temkinli davranıp beyan etmek.
2. **Device ID**: Crashlytics Installation UUID ve Firebase installation ID'nin Apple'ın "Device ID" tanımına
   girmediği varsayıldı. Apple incelemesi farklı yorumlarsa eklenmeli.
3. Yayın ikilisinde `AdSupport.framework` bağlı olmadığı (RevenueCat ve Firebase sürümleri: 5.94.0 / 12.19.2)
   derlenmiş uygulamada kontrol edilmeli.
4. App Store Connect'teki mevcut beyanlar okunup bu tabloyla birleştirilmeli (ör. Purchase History zaten var mı).

Kaynaklar: [Firebase gizlilik ve Crashlytics saklama](https://firebase.google.com/support/privacy) ·
[Firebase – App Store veri açıklaması](https://firebase.google.com/docs/ios/app-store-data-collection) ·
[RevenueCat gizlilik politikası](https://www.revenuecat.com/privacy) · [RevenueCat DPA](https://www.revenuecat.com/dpa)

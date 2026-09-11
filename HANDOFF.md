# Clockin iPhone — devir notu

Bu klasör Clockin'in iPhone sürümü için. Mac uygulaması üzerinde uzun bir
çalışma oturumundan ayrıldı; buradaki bilgiler o oturumda doğrulandı.

## Kaynak proje

- Mac uygulaması: `../clockin-main`
- Upstream repo: `ismailakdag/clockin` (sahibi İsmail)
- Fork: `erdmncdr/clockin` — katkılar buradan PR olarak gidiyor
- Git kimliği: `erdmncdr` / `edolin67@gmail.com`

İsmail'in reposunun yapısı onun onayı olmadan değiştirilmemeli. Bu yüzden
iPhone işi şimdilik bu ayrı klasörde.

## Taşınabilirlik (Mac kodu üzerinde yapılan tarama)

26 Swift dosyasının 20'si AppKit / pencere / menü çubuğu API'si kullanmıyor:

`Models`, `CSVImporter`, `PastedTextImporter`, `ImportComparison`,
`ExchangeRates`, `RadioController`, `Themes`, `ButtonStyles`, `MainTabBar`,
`HistoryView`, `HeatmapView`, `ProgressView`, `ManualEntryView`,
`ManualStartView`, `RateScheduleView`, `ImportComparisonView`,
`SessionSummaryView`, `GuideView`, `UIScale`, `UpdateChecker`

Uyarı: tarama sembol aramasıydı, derleme değil. iOS'ta derlenmeden taşınabilir
sayılmamalı.

iOS'ta anlamı olmayanlar: `UIScale` (iOS'ta Dynamic Type kullanılır),
`UpdateChecker` (App Store / TestFlight kurulumunda gereksiz).

- `ClockStore` AppKit'e **tek satırla** bağlı: `setPinned` içinde
  `PinnedWindowController.shared.update(...)`. O çağrı ayrılınca mağaza iki
  platformda da kullanılabilir.
- macOS'a özel, yeniden yazılması gerekenler: `ClockinApp` (menü çubuğu),
  `MainWindow`, `PinnedWindow`, `KeyboardShortcuts`, `FocusChime` (NSSound),
  `MascotAsset` (NSImage), panel/pano kullanan kısımlar (`PasteImportView`,
  `SettingsView`, `ShareStatsView`).

## Ortam

- Xcode kurulu ve seçili (`xcode-select` Xcode'u gösteriyor).
- iOS 26.5 SDK ve simülatörler var (iPhone 17 Pro, 17 Pro Max, 17e).
- Yalnızca Command Line Tools ile SwiftUI derlenmiyor (SwiftUI makro eklentisi
  Xcode ile geliyor). Xcode seçili kalmalı.

## Bekleyen kararlar

1. **Senkronizasyon.** Mac verisi yerel JSON:
   `~/Library/Application Support/Clockin/clockin.json`. iPhone ayrı dosya
   tutarsa iki ayrı kayıt olur (telefonda başlayan sayacı Mac bilmez, toplamlar
   ayrışır). Gerçekçi çözüm iCloud, ama iCloud yetkisi **ücretli Apple Developer
   hesabı** (99 $/yıl) istiyor; ücretsiz Apple ID ile imzalanan uygulama
   iCloud kullanamıyor.
2. **Dağıtım.** Simülatör ücretsiz. Ücretsiz Apple ID ile kendi iPhone'a kurulum
   7 günde bir yenilenmeli. Başkasıyla paylaşmak (TestFlight / App Store) ücretli
   hesap istiyor.
3. **İsmail.** Ortak `ClockinCore` modülü ve iOS hedefi upstream repoya girecekse
   önce onunla konuşulmalı.

## Plan

1. Prototip, bu klasörde: taşınabilir kodu kopyala (İsmail'in reposuna
   dokunmadan), SwiftUI iOS uygulaması, yerel veri. Kapsam: sayaç, bugün, son
   kayıtlar, elle kayıt ekleme.
2. Simülatörde çalıştırıp dene.
3. Karar sonrası: iCloud senkronizasyonu, Live Activity / Dynamic Island
   (çalışan sayaç ve kazanç), ana ekran widget'ı, App Intents / Shortcuts ile
   clock in. CSV içe aktarma, heatmap, ücret takvimi başta Mac'te kalabilir.

## Mac tarafında öğrenilen tuzaklar

- `money()` `FormatStyle` kullanıyor; `maxFractionDigits: 0` ile
  `.fractionLength(2...0)` geçersiz aralık oluşturup çöküyordu. Doğrusu
  `minimum = min(2, maximum)`. Regresyon testi `Tests/manual/main.swift`'te.
- Picker seçimini `Double` eşitliğine bağlamak kırılgan (32 bit float ile yazılan
  1.2999999523 hiçbir etiketle eşleşmiyor, Picker boş görünüyor). Tam sayı kullan.
- `.buttonStyle(.plain)` tıklama alanını çizilen piksellere indiriyor;
  `contentShape(Rectangle())` gerekli.
- Görünüm gövdesinde tekrar tekrar okunan hesaplanmış özellikler her okumada
  baştan hesaplanıyor; toplamlar ve günlük değerler mağazada önbelleklenmeli.
- Kayıtları gün bazında birleştirme; CSV tekilleştirmesi başlangıç + bitiş + süre
  üçlüsüne dayanıyor.

## Çalışma tarzı

- Kullanıcı Türkçe yazıyor.
- Commit mesajları İngilizce ve "neden" odaklı.
- Kullanıcı arayüzü kendisi gözle kontrol ediyor; görsel doğrulama yapılamadıysa
  bu açıkça söylenmeli.

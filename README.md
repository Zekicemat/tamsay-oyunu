# LlenO - Matematiksel Kart Oyunu

Bu proje, React Native ve Expo kullanılarak geliştirilmiş, UNO benzeri ancak matematiksel işlem ve kelime bilgisi kuralları içeren sıra tabanlı bir mobil kart oyunudur.

## 🚀 Özellikler

*   **Çok Oyunculu (Online):** Firebase Realtime Database üzerinden arkadaşlarınızla gerçek zamanlı oynayabilirsiniz.
*   **Tek Oyunculu (Bot):** Yapay zekaya (Bot) karşı internet bağlantısı olmadan oynayabilirsiniz.
*   **Matematiksel Oyun Modları:** Toplama (+), Çıkarma (-), Çarpma (x), Bölme (/) ve Sıralama (Büyüklük) modları.
*   **Eğitici Ceza Sistemi:** Kart çeken oyuncuya İngilizce kelime sorusu sorulur, bilirse kartı (desteye geri koyar).
*   **Joker Kartlar:** Yön Değiştirme, +2, +4, İşaret Değiştirici gibi stratejik kartlar.
*   **Modern Arayüz:** Animasyonlu kartlar, sürükle-bırak desteği ve kullanıcı dostu arayüz.

## 🛠 Kullanılan Teknolojiler

*   **React Native** (Expo SDK 52+)
*   **React Navigation v7** (Ekran geçişleri)
*   **Firebase** (Realtime Database - Online oyun senkronizasyonu)
*   **Expo Clipboard** (Oda kodu kopyalama)
*   **React Native Reanimated & Moti** (Kart animasyonları)
*   **React Native Safe Area Context** (Çentikli ekran uyumluluğu)

## 🏁 Kurulum ve Çalıştırma

Projeyi yerel ortamınızda çalıştırmak için aşağıdaki adımları izleyin.

### 1. Gereksinimler
*   Node.js (LTS sürümü önerilir)
*   npm veya yarn
*   Expo Go uygulaması (Telefonda test etmek için) ve Android Studio (bilgisayarda test edip(emülatör ile) uygulamayı geliştirmek için)

### 2. Bağımlılıkları Yükleyin
Proje dizinine gidin ve paketleri yükleyin:

```bash/terminal
cd uno-mobil-app
npm install
```

### 3. Uygulamayı Başlatın
Geliştirme sunucusunu başlatmak için:

```bash
npx expo start
```

*   **Fiziksel Cihaz:** Terminalde çıkan QR kodu telefonunuzdaki **Expo Go** uygulaması ile taratın.
*   **Android Emülatör:** `a` tuşuna basın.



```bash
npx expo start -c


bunu yazarak projeyi expo ya yükleriz ve telefonda indirilip oynanabilir hale getiririz.
eas build -p android --profile production

```

## 🔥 Firebase Yapılandırması

Proje, backend olarak Firebase kullanır. Online oyunun çalışması için `src/config/firebase.js` dosyasının doğru yapılandırılmış olması gerekir.

**Veritabanı Yapısı (Realtime DB):**
*   `rooms/{roomCode}`: Her oyun odası için bir düğüm oluşturulur.
*   `gameState`: Oyunun anlık durumu (kartlar, sıra, puanlar vb.) burada tutulur.

**Not:** Proje şu an anonim kimlik doğrulama (Anonymous Auth) kullanmaktadır. Firebase Console üzerinden Authentication > Sign-in method > Anonymous seçeneğinin aktif olduğundan emin olun.

## 🏗 Proje Mimarisi

Proje modüler bir yapıda geliştirilmiştir. Yeni bir geliştirici için önemli klasörler:

*   **`src/game/engine.js` (Oyun Motoru):**
    *   Tüm oyun mantığı, kurallar, kart dağıtımı ve tur sonuçlandırma işlemleri burada yapılır.
    *   `GAME_PHASES`: Oyunun durum makinesi (SELECTION -> REVEAL -> CALCULATION -> ROUND_RESULT vb.).
    *   `createNewGame`, `toggleCardSelection`, `resolveRound` gibi temel fonksiyonlar buradadır.

*   **`src/screens/` (Ekranlar):**
    *   `GameScreen.js`: Oyunun oynandığı ana ekran. UI ve Oyun Motoru arasındaki köprüdür. Bot mantığı (`useEffect` hook'ları) burada tetiklenir.
    *   `LobbyScreen.js`: Oda oluşturma/katılma işlemleri.
    *   `HowToPlayScreen.js`: Oyun kuralları.

*   **`src/services/` (Servisler):**
    *   `roomService.js`: Firebase ile iletişim kuran, oda oluşturan ve veriyi dinleyen servis.
    *   `nickname.js`: Oyuncu adını yerel depolamada (AsyncStorage/Context) tutar.

*   **`src/ui/` (Bileşenler):**
    *   `CardView.js`: Tek bir oyun kartının görseli.
    *   `CardView.js`: Tek bir oyun kartının görseli..
    *   `QuizModal.js`: Ceza sorularının gösterildiği pencere.
    *   `CalculationModal.js`: İşlem sorularının (hesaplama) sorulduğu pencere.
    *   `ResultOverlay.js`: Tur sonucu animasyonu.

*   **`src/data/`:**
    *   `cardData.js`: İngilizce-Türkçe kelime havuzu.

## 🐛 Hata Ayıklama ve Geliştirme İpuçları

1.  **Oyun Akışı:**
    *   Oyun `SELECTION` fazında başlar.
    *   Her iki oyuncu kart seçip onaylayınca `REVEAL` fazına geçilir.
    *   Sonuç hesaplanır, eğer bir işlem sorusu gerekiyorsa `CALCULATION` fazı başlar.
    *   Sonuç gösterilir (`ROUND_RESULT`), ardından ceza varsa kart çekilir (`VOCAB_CHECK`).
    *   Elindeki kartları bitiren kazanır.

2.  **Tanımsız Durumlar (0'a Bölme):**
    *   Oyun motoru (`engine.js`), 0'a bölme gibi durumlarda `resultValue`'yu `0` yapar ancak `isUndefined` bayrağını `true` set eder.
    *   Bu durumda tur nötr sayılır, kimse kart çekmez ve `QuizModal` açılmaz.

3.  **Bot Mantığı:**
    *   Bot sadece `isOnline: false` durumunda aktiftir.
    *   `GameScreen.js` içindeki `useEffect` hook'ları, sıra botta olduğunda `setTimeout` ile yapay bir gecikme ekleyerek `engine.js` fonksiyonlarını çağırır (kart seçme, cevap verme vb.).# tamsay-oyunu

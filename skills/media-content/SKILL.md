---
name: media-content
description: Sosyal medya içerik üretim standartları (Instagram Reels, post, story, kısa video). Kullanıcı sosyal medya içeriği, Reels senaryosu, video konsepti, içerik takvimi, story serisi, AI video üretimi (Higgsfield vb.) veya tanıtım videosu planlamak/üretmek istediğinde MUTLAKA bu skill'i kullan. "Reels yap", "video fikri", "içerik planı", "sosyal medya postu", "UGC video", "tanıtım videosu" gibi ifadeler geçtiğinde de kullan. Platform kurallarına uygun, marka tutarlı, üretime hazır içerik çıkarır.
---

# Media Content

Sosyal medya içerik üretimi (planlama → senaryo → üretim → yayın) standartları. Projeye özel detaylar (marka, ton, palet, fontlar, hedef kitle, yayın sıklığı) CLAUDE.md'dedir. Metin gövdesi (caption, başlık) yazımında `copywriting` skill kuralları geçerlidir; bu skill görsel/video içeriğin kendisine ve üretim sürecine odaklanır.

## İçerik türü seçimi

Amaca göre tür öner; kullanıcı tür belirttiyse ona uy:

- **Erişim/keşfet**: Reels (kısa video) — algoritmanın en çok dağıttığı format.
- **Güven/vitrin**: carousel post (ürün detayı, öncesi-sonrası, ipucu serisi).
- **Sıcaklık/aciliyet**: story (perde arkası, anket, geri sayım) — 24 saatlik, samimi ton.
- **Otorite**: tek görsel + güçlü caption (veri, analiz, duyuru).

## Reels / kısa video standartları

- **Süre**: 7-20 saniye ideal; 30 saniyeyi yalnızca anlatı gerektiriyorsa aş.
- **İlk 1-2 saniye kanca**: hareket, iddia veya soru — logo açılışıyla BAŞLAMA, izleyici kaydırır.
- **Format**: 9:16 dikey, 1080x1920. Güvenli alan: alt %15 ve üst %10'a kritik metin koyma (UI elemanları örter).
- **Metin bindirme**: sessiz izlenmeye hazır olmalı (izleyicilerin çoğu sessiz izler) — konuşma varsa altyazı, yoksa ekran metniyle mesaj taşınır. Metin kısa ve büyük; karede en fazla 1-2 satır.
- **Senaryo yapısı**: kanca (0-2sn) → gelişme/gösterim (orta) → sonuç + CTA (son 2-3sn). Senaryoyu saniye aralıklı sahne listesi olarak yaz.
- **Ses**: platform içi trend ses kullanılacaksa yayın anında seçilir; senaryoda "trend ses / sakin fon" gibi tip belirt.

## Görsel üretim kuralları

- Marka paleti ve fontlar her içerikte tutarlı (değerler CLAUDE.md'de). Şablonlaşabilir öğeler (kapak düzeni, alt bant, logo konumu) bir kez tanımlanıp tekrar kullanılır.
- Kapak görseli (Reels cover / ilk carousel karesi) tek başına anlamlı olmalı: net başlık + görsel — profil ızgarasında da okunur.
- Fotoğraf seçiminde: gerçek/doğal kareler stok görünümlü karelere tercih edilir; yeme-içme içeriğinde doğal ışık ve yakın çekim.
- Boyutlar: post 1080x1350 (4:5, akışta daha fazla alan), carousel kareleri aynı boyutta, story/Reels 1080x1920, kare post 1080x1080, OG image 1200x630, YouTube kapak 1280x720. **Platform standart boyutlarının tek kaynağı bu skill'dir**; CLAUDE.md yalnızca marka-özel istisnaları tutar, `media-editing` bu boyutları uygular.

## AI video üretimi (Higgsfield vb.)

- AI üretim akışı: önce senaryo ve sahne listesi netleşir → sahne başına prompt yazılır → üretilen kesitler birleştirilir. Prompt'suz "bir video yap" ile araca gitme.
- Sahne prompt'u şablonu: [çekim tipi] + [özne ve eylem] + [ortam/ışık] + [kamera hareketi] + [stil]. Türkçe metin bindirmeleri AI'a yaptırma — metin bindirme montajda eklenir (AI üretimli Türkçe yazı hatalı çıkar).
- Marka yüzü/ürün gerçekse: gerçek çekim + AI b-roll karışımı öner; tamamen sentetik içerikte markanın gerçek görselleriyle tutarlılığı kontrol et.
- Programatik üretim tekrarlanan şablon işlerde (fiyat duyurusu, haftalık menü gibi) AI'dan önce değerlendirilir — deterministik, ücretsiz, marka fontlarıyla birebir. Üretimin tekniği (araç seçimi, bindirme, encode ayarları) `media-editing`'e tabidir.

## İçerik planı / takvim

- Plan istendiğinde çıktı tablo halinde: tarih, tür, konu/kanca, CTA, üretim notu.
- Karışım kuralı (aksi belirtilmedikçe): %40 değer/eğitim, %30 vitrin/ürün, %20 sosyal kanıt/perde arkası, %10 doğrudan satış. Arka arkaya iki satış içeriği koyma.
- Yayın sıklığında sürdürülebilirlik > yoğunluk: kullanıcının üretim kapasitesini sor, ona göre plan (haftada 3 kaliteli > her gün zayıf).

## Yayın kontrol listesi

- [ ] Kapak tek başına anlamlı ve okunaklı
- [ ] Sessiz izlemede mesaj anlaşılıyor
- [ ] Güvenli alanlarda kritik metin yok
- [ ] Caption `copywriting` kurallarına uygun (kanca ilk cümlede, tek CTA, 3-8 hashtag)
- [ ] Marka paleti/font tutarlı
- [ ] Telifli müzik/görsel ticari hesapta kullanılmıyor (platform içi lisanslı sesler hariç)

## Çıktı formatı

- Senaryo: saniye aralıklı sahne tablosu (sahne, görüntü, ekran metni, ses notu).
- Tek içerik talebi: konsept (1-2 cümle) + senaryo/görsel tarifi + caption + üretim notları.
- Plan talebi: takvim tablosu + ilk içeriğin tam açılımı örnek olarak.

## İçerik stratejisi ve konu haritası

Tekil içerik üretiminden önce strateji katmanı — plan yoksa üretim başlamaz:

- **İçerik sütunları**: marka başına 3-5 sütun tanımlanır (ör. bir restoran markası: tarif, teknik ipucu, mekân, sezon) ve her sütunun amacı bellidir (keşif / güven / dönüşüm). Sütun dışı rastgele içerik üretilmez.
- **Konu haritası**: sütun × format matrisi (Reels/carousel/story) doldurulur; her konu hedef izleyici sorusuna bağlanır ("bunu kim neden izler/kaydeder?"). Cevabı olmayan konu listeden düşer.
- **Takvim disiplini**: sürdürülebilir frekans belirlenir (haftada X) ve takvim en az 2 hafta ileriyi gösterir; "bugün ne paylaşsak" günü yaşanmaz. Takvim, approval-workflow taslak hattıyla uyumlu üretilir (üretim → onay → yayın tarihleri ayrı).
- **Geri besleme döngüsü**: aylık gözden geçirmede sütun bazında performans (kaydetme/paylaşım öncelikli metrikler) değerlendirilir; işlemeyen sütun revize edilir veya emekli edilir — hissiyatla değil veriyle.

## Reklam kreatif varyasyon üretimi

Organik içerikten farklı olarak reklam kreatifi sistematik varyasyonla üretilir (strateji ve bütçe kararları ad-campaign-management'a tabidir — bu bölüm kreatifin kendisini üretir):

- **Varyasyon matrisi**: kanca (3 farklı açı) × format (görsel/video/carousel) × CTA — tek "en iyi tahminimiz" kreatif yerine test edilebilir küçük set (4-6 varyant) üretilir.
- **Kanca açıları**: problem-öncelikli / sonuç-öncelikli / sosyal kanıt-öncelikli / itiraz-kırıcı — aynı mesajın farklı giriş kapıları; copywriting davranışsal tetikleyici kuralları (gerçeklik şartı) burada da geçerlidir.
- **İlk 2 saniye kuralı**: video reklamda kanca sessiz izlemede de çalışmalı (altyazı/görsel kanca); logo ile açılan reklam üretilmez.
- **Yorgunluk yönetimi**: kazanan kreatifin varyasyonları hazırda bekletilir; frekans yükselince (ad-campaign-management metrik tablosuna göre) taze varyant devreye girer — sıfırdan üretim krizine düşülmez.
- **Uyum**: platform reklam politikaları (yanıltıcı iddia, önce/sonra sağlık vaadi vb.) üretim aşamasında kontrol edilir; reddedilecek kreatif hiç üretilmez.

---
name: pricing
description: SaaS paketleme ve fiyatlandırma stratejisi standartları — plan yapısı, değer metriği seçimi, fiyat kademeleri, Türkiye pazarına uyarlama, fiyat değişikliği yönetimi. Kullanıcı fiyatlandırma, paket tasarımı, plan kademeleri, ücretsiz/deneme stratejisi, fiyat artışı veya "ne kadar ücret alayım" tipi bir soru sorduğunda MUTLAKA bu skill'i kullan. "Fiyat belirle", "paketleri tasarla", "freemium mı trial mı", "fiyat sayfası", "zam yapacağım", "monetizasyon" gibi ifadeler geçtiğinde de kullan. Teknik plan/abonelik şeması saas-patterns'a tabidir — bu skill fiyat ve paket kararının kendisini taşır.
---

# Pricing (Paketleme ve Fiyatlandırma)

SaaS ürünlerde fiyat ve paket kararı standartları. Plan tablolarının veritabanı yapısı saas-patterns'a, fiyat sayfası metni copywriting'e, dönüşümü cro'ya aittir.

## Temel ilkeler

1. **Değer metriği önce**: Fiyat, kullanıcının aldığı değerle büyüyen tek bir metriğe bağlanır (öğrenci sayısı, tenant sayısı, aylık içerik adedi, hesaplama sayısı). Değerle ilgisi olmayan metriğe (ör. "kullanıcı başına" ama değeri kayıt sayısı üretiyorsa) fiyat bağlanmaz.
2. **3 kademe varsayılan**: Giriş / Önerilen / Üst. 4+ kademe karar felcine yol açar; tek kademe pazarı daraltır. Önerilen paket görsel olarak vurgulanır ve gerçekten çoğunluğa uyar.
3. **Kademeler arası fark net**: Her üst kademe, alt kademedeki somut bir sınıra çarpan kullanıcının doğal sonraki adımıdır. "Öylesine ekstra özellik" ile kademe doldurulmaz.
4. **Fiyat uydurulmaz, sınanır**: İlk fiyat rakip/ikame maliyet analizi + hedef müşterinin ödediği mevcut alternatifle gerekçelendirilir; sonra gerçek satış görüşmelerinden gelen sinyalle ayarlanır. "İçime öyle doğdu" fiyatı yasak.

## Türkiye pazarı uyarlamaları

- TL fiyatlamada enflasyon gerçeği: yıllık sözleşmelerde fiyat güncelleme maddesi baştan tanımlanır; döviz endeksli fiyat kullanılacaksa sözleşmede açık yazılır.
- KDV dahil/hariç gösterimi hedef kitleye göre: B2C'de KDV dahil, B2B'de hariç + açık ibare. Belirsiz bırakılmaz.
- Yıllık ödeme indirimi (%15-20 aralığı makul) nakit akışı aracı olarak sunulur; aylıktan caydırma cezası gibi kurgulanmaz.

## Ücretsiz katman kararı

- **Trial (süreli deneme)**: Değeri hızlı kanıtlanan ürünlerde varsayılan (14 gün, kredi kartı istemeden başla).
- **Freemium**: Yalnızca ücretsiz kullanıcının maliyeti düşük VE ağ etkisi/yayılım değeri varsa. "Belki dönüşür" umuduyla freemium açılmaz.
- Ücretsiz katman sınırı, değer metriğinin küçük ama gerçek bir dilimi olur; sakat bırakılmış ürün ("export yok") yerine hacim sınırı tercih edilir.

## Fiyat değişikliği yönetimi

- Mevcut müşteriye zam: en az 30 gün önceden, gerekçeli, kişisel bildirimle; mümkünse mevcutlara geçiş dönemi (grandfathering) tanımlanır.
- Fiyat değişikliği bir migration gibi ele alınır: etkilenen müşteri listesi, iletişim planı, geri dönüş (rollback) kararı önceden yazılır.
- İndirim disiplini: pazarlıkla verilen her özel fiyat kayıt altına alınır ve bitiş tarihi olur; süresiz özel fiyat birikimi yasak.

## Sınırlar

- Aldatıcı fiyatlandırma yasak: gizli zorunlu ek ücret, otomatik yenilemenin saklanması, iptalin zorlaştırılması. Abonelik/iptal hukuku metinleri kvkk-legal kapsamındadır.

## Çıktı formatı

Paket tablosu (kademe | değer metriği sınırı | fiyat | hedef müşteri) + her kademenin gerekçesi (1 cümle) + fiyat varsayımlarının kaynağı (rakip / ikame maliyet / müşteri görüşmesi) + gözden geçirme tarihi.

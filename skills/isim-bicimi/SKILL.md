---
name: isim-bicimi
description: Türkçe isim alanı biçimlendirme standardı — kişi/kurum/ürün adı girilirken her kelimenin ilk harfi büyük (Title Case, tr-TR), açıklama/not/yorum gibi serbest metin alanlarına DOKUNULMAZ. Kullanıcı ad soyad girişi, hasta/personel/müşteri/tedarikçi adı, form alanı, isim normalizasyonu, "ilk harfler büyük olsun", "ad yazarken düzeltsin", İ/ı büyük-küçük harf hatası veya mevcut isimleri toplu düzeltme istediğinde MUTLAKA bu skill'i kullan. "Ad soyad alanı", "isimleri düzelt", "ilk harf büyük", "açıklamaya karışma", "KEMAL yazılmış Kemal olsun" gibi ifadeler geçtiğinde de kullan. Form deseni code-standards'a, doğrulama security-baseline'a, şema sql-migration'a tabidir — bu skill hangi alanın biçimlendirileceğini ve Türkçe harf kurallarını taşır.
---

# İsim Biçimi

**Kural:** İsim alanlarında her kelimenin ilk harfi büyük, gerisi küçük saklanır ("AYŞE nur" → "Ayşe Nur"). **Açıklama alanlarına asla uygulanmaz** — kullanıcı ne yazdıysa o saklanır.

## 1. Hangi alan hangi grupta

Alan eklerken grup bilinçli seçilir ve CLAUDE.md'deki alan listesinde tutulur.

| Grup | Örnek alanlar | Davranış |
|---|---|---|
| **İsim** | `ad_soyad`, personel/terapist/hasta adı, cari/tedarikçi adı, kurum adı, tedavi/paket/oda/cihaz adı, etiket adı | `isimBicimi()` uygulanır |
| **Açıklama (dokunulmaz)** | `aciklama`, `not`, `yorum`, `sikayet`, `anamnez`, mesaj/şablon metni, `adres_detay`, iptal/red gerekçesi | Olduğu gibi; yalnızca `trim()` |
| **Biçimi başka kuralla** | e-posta (küçük harf), kullanıcı adı, TC kimlik, IBAN, telefon, il/ilçe/mahalle (`adres-girisi`: paketten geldiği gibi) | `isimBicimi()` uygulanmaz |

Belirsiz alanda varsayılan: **dokunma**. Yanlışlıkla büyük harfe çevrilmiş bir açıklama, yanlışlıkla küçük bırakılmış bir isimden daha zararlıdır. Kısaltma içeren ad alanlarında ("MR Cihazı", "SGK") `kisaltmaKoru` seçeneği kullanılır (bkz. aşağıda).

## 2. Fonksiyon (tek yerde, `lib/isim.ts`)

```ts
const TR = "tr-TR";

export function isimBicimi(girdi: string, secenek?: { kisaltmaKoru?: boolean }): string {
  return girdi
    .trim()
    .replace(/\s+/g, " ")
    .split(" ")
    .map((kelime) =>
      secenek?.kisaltmaKoru && /^\p{Lu}{2,4}$/u.test(kelime)
        ? kelime
        : kelime
            .toLocaleLowerCase(TR)
            .replace(/(^|[-'’])(\p{L})/gu, (_, ayirac, harf) => ayirac + harf.toLocaleUpperCase(TR)),
    )
    .join(" ");
}
```

Doğrulanmış örnekler (test dosyasına aynen girer):

| Girdi | Çıktı |
|---|---|
| `AYŞE nur` | `Ayşe Nur` |
| `IŞIL`, `ışıl` | `Işıl` |
| `İBRAHİM`, `ibrahim` | `İbrahim` |
| `  ali    veli ` | `Ali Veli` |
| `mehmet-ali` | `Mehmet-Ali` |
| `o'neil` | `O'Neil` |
| `ÇAĞLA ÖZGÜR` | `Çağla Özgür` |
| `""` | `""` |

**Türkçe harf tuzağı:** `toLowerCase()`/`toUpperCase()` kullanılmaz — `I→i`, `i→I` eşlemesi yapar (`IŞIL` → `ışıl` yerine `işil`, `ibrahim` → `Ibrahim`). Her ikisi de `toLocale…Case("tr-TR")` olmalıdır. Node'da tam ICU gerekir (Node ≥ 13 varsayılan).

## 3. Nerede uygulanır

1. **Sunucu doğrulama katmanı yetkilidir** (security-baseline: client güvenlik değildir). Zod şemasında:

```ts
ad_soyad: z.string().trim().min(2, "Ad soyad en az 2 karakter olmalı.").transform((s) => isimBicimi(s)),
aciklama: z.string().trim().optional(), // dokunulmaz
```

2. **Form alanı, `onBlur`'da** biçimlendirir ve kullanıcıya düzeltmeyi gösterir. `onChange`'de biçimlendirilmez: imleç zıplar, "ali " yazarken boşluk yenir, "Mc" gibi ara durumlar bozulur.
3. **Toplu giriş yolları da aynı fonksiyondan geçer:** arşiv/Excel içe aktarma, QR/portal kayıt formu, başvuru formu, API route. Tek giriş yolu unutulursa tutarsız veri oluşur — yeni isim alanında tüm yazma yollarını tara (`grep -rn "ad_soyad" app lib`).
4. **Görüntüleme katmanı biçimlendirmez.** Veri doğru saklanır, olduğu gibi gösterilir. CSS `capitalize` çözüm değildir: yalnızca görünümü değiştirir, veriyi değil; `lang="tr"` yoksa İ/ı'yı yanlış çevirir; aramada/dışa aktarmada eski ham hali çıkar.

## 4. Arama ve eşleştirme

- Karşılaştırma büyük/küçük harften bağımsız ve Türkçe kurallı yapılır: `a.toLocaleLowerCase("tr-TR").includes(q.toLocaleLowerCase("tr-TR"))`. DB'de `ilike` harf eşleşmesinde Türkçe İ/ı'yı doğru eşlemeyebilir; kritik aramalarda normalize edilmiş arama kolonu veya `unaccent`/lower stratejisi tasarlanır (sql-queries).
- Tekillik (aynı isimli kişi/oda) kontrolü biçimlendirilmiş değer üzerinden yapılır; "ayşe nur" ve "AYŞE NUR" aynı kayıttır.

## 5. Mevcut veriyi düzeltme (backfill)

- **Postgres `initcap()` kullanılmaz:** DB locale'ine bağlıdır, `ibrahim` → `Ibrahim`, `ışıl` → `Işıl` dışındaki tuzaklarda güvenilmezdir.
- Yöntem: TS script'i `isimBicimi`'yi kullanarak tabloyu okur, **değişecek satırları önce listeler** (eski → yeni, sayı), onaydan sonra idempotent `UPDATE` yapar. Toplu veri değişikliği sql-migration kuralı: geri dönüş için yedek tablo, etkilenen satır sayısı önceden.
- Kural gereği bilinçli kalan biçimler (ör. "Ahmet bin Ali" gibi parçacık tercihi) varsa listede insan kontrolüne bırakılır.

## 6. Kapsam dışı ve sınırlar

- Parçacık istisnası (bin, van, von, de) **varsayılan olarak yoktur**: her kelimenin ilk harfi büyük kuralı basit ve öngörülebilir tutulur. Kurum isterse CLAUDE.md'de istisna listesi tanımlar.
- Çok harfli özel yazımlar (McDonald, DiCaprio) otomatik yakalanmaz ve sunucu her kayıtta kuralı yeniden uyguladığı için elle düzeltmeler de ezilir (`McDonald` → `Mcdonald`). Bu bilinçli bir takastır: basit ve öngörülebilir kural, nadir istisnadan önce gelir. İstisna gerekirse CLAUDE.md'de alan bazlı `isimBicimi` atlama listesi tanımlanır.
- Tamamen büyük harfle yazılması gereken kimlik alanları (plaka, belge no) bu skill'in konusu değildir.

## Kontrol listesi

- [ ] Alan "isim" mi "açıklama" mı, listede yazılı mı?
- [ ] Zod `transform` sunucuda var mı; açıklama alanında yok mu?
- [ ] Form `onBlur`'da biçimlendiriyor, `onChange`'de değil mi?
- [ ] Tüm yazma yolları (içe aktarma, QR form, portal, API) aynı fonksiyondan geçiyor mu?
- [ ] `toLocaleLowerCase/UpperCase("tr-TR")` kullanıldı mı, çıplak `toLowerCase/toUpperCase` yok mu?
- [ ] Test: tablodaki 8 örnek (özellikle `IŞIL`, `İBRAHİM`) geçiyor mu?
- [ ] Mevcut veri için backfill önce liste olarak gösterildi mi?

## Çıktı formatı

Kod dosya yollarıyla: `lib/isim.ts` + testi, Zod şema değişikliği, form `onBlur` bağlantısı. Backfill istendiyse: önce etkilenecek satır sayısı ve eski→yeni örnek listesi, sonra idempotent güncelleme.

## assignment__change
# Tayin alanının doldurulması

# 1120 BTE – Uygulama Adımları Rehberi

> **Kapsam:** FI Belge Posting – ZUONR Otomatik Doldurma  
> **İlgili Nesne:** BTE 1120 / BSEG_SUBST Append Structure

---

## Adım 1 – FIBF'te BTE'yi Zle (Uyarlama)

1. Transaction: **FIBF** aç.
2. Menüden **Settings → Products → Of a Customer** seçeneğine git.
3. Yeni bir ürün tanımla (yoksa) veya mevcut ürünü seç.
4. Menüden **Settings → P/S Function Modules → Of a Customer** seçeneğine git.
5. Aşağıdaki değerleri girerek kaydet:

| Alan | Değer |
|---|---|
| Event | `00001120` |
| Function Module | `Z_FI_BTE_1120` *(oluşturulacak FM adı)* |
| Product | *(tanımladığın ürün)* |

> ✅ Bu adımdan sonra SAP, FI belgesi kaydedildiğinde otomatik olarak bu FM'i çağıracaktır.

---

## Adım 2 – BSEG_SUBST için Append Structure Oluştur

BTE 1120, `BSEG_SUBST` yapısını kullanır. Özel alanlar eklemek için append structure oluşturulması gerekir.

1. Transaction: **SE11** aç.
2. **Structure** alanına `BSEG_SUBST` yaz → **Display** ile aç.
3. Menüden **Extras → Append Structure → Create** seçeneğine tıkla.
4. Append structure adını gir: **`ZFI_000_BSEG_SUBST`**
5. **Components** sekmesine geç ve aşağıdaki alanları ekle:

| Component | Typing Method | Component Type | Data Type | Length | Decimals | Kısa Tanım |
|---|---|---|---|---|---|---|
| `HKONT` | Types | `HKONT` | CHAR | 10 | 0 | Defter kebiri hesabı |
| `ACROBJ_ID` | Types | `ACR_OBJ_ID` | CHAR | 32 | 0 | Dönemselleştirme nesnesi |
| `DMBTR` | Types | `DMBTR` | CURR | 23 | 2 | Ulusal para birimi tutarı |
| `DMBE2` | Types | `DMBE2` | CURR | 23 | 2 | İkinci ulusal para birimi tutarı |
| `DMBE3` | Types | `DMBE3` | CURR | 23 | 2 | Üçüncü ulusal para birimi tutarı |
| `WRBTR` | Types | `WRBTR` | CURR | 23 | 2 | Belge para birimi tutarı |

6. **Kaydet** ve **Aktive et** (Activate).

> ✅ Append structure aktive edildikten sonra `BSEG_SUBST` yapısı bu alanları içerecektir.

---

## Adım 3 – Function Module Oluştur (SE37)

1. Transaction: **SE37** aç.
2. FM adını gir: `Z_FI_BTE_1120` → **Create**.
3. Mevcut BTE 1120 arayüzünü baz alarak parametreleri tanımla.
4. Aşağıdaki kodu FM'in kaynak koduna yaz:

```abap
LOOP AT t_bsegsub REFERENCE INTO DATA(lr_bsegsub)
  WHERE hkont+00(03) EQ '646'
     OR hkont+00(03) EQ '656'.

  LOOP AT t_bseg WHERE koart EQ 'K'
                    OR koart EQ 'D'
                    OR koart EQ 'S'
                   AND hkont+00(03) NE '646'
                   AND hkont+00(03) NE '656'.

    READ TABLE t_bsegsub ASSIGNING FIELD-SYMBOL(<bsegsub>)
      WITH KEY tabix = sy-tabix.

    IF sy-subrc IS INITIAL.
      CASE t_bseg-koart.
        WHEN 'K'. <bsegsub>-zuonr = |K-{ t_bseg-lifnr }|.
        WHEN 'D'. <bsegsub>-zuonr = |D-{ t_bseg-kunnr }|.
        WHEN 'S'. <bsegsub>-zuonr = |S-{ t_bseg-hkont }|.
      ENDCASE.
    ENDIF.

    EXIT.
  ENDLOOP.

  CASE t_bseg-koart.
    WHEN 'K'. lr_bsegsub->zuonr = |K-{ t_bseg-lifnr }|.
    WHEN 'D'. lr_bsegsub->zuonr = |D-{ t_bseg-kunnr }|.
    WHEN 'S'. lr_bsegsub->zuonr = |S-{ t_bseg-hkont }|.
  ENDCASE.

ENDLOOP.
```

5. **Kaydet** ve **Aktive et**.

---

## Adım 4 – Test Et

1. Transaction: **FB01** veya **F-02** ile test belgesi oluştur.
2. Belge satırlarından en az biri **646** veya **656** ile başlayan hesap kodu içermeli.
3. Belgeyi kaydet.
4. Kaydedilen belgeyi **FB03** ile aç → ilgili satırın **ZUONR** alanının dolduğunu doğrula.

| Beklenen ZUONR Değeri | Koşul |
|---|---|
| `K-{LIFNR}` | Karşı kalem tipi Satıcı (K) |
| `D-{KUNNR}` | Karşı kalem tipi Müşteri (D) |
| `S-{HKONT}` | Karşı kalem tipi G/L Hesabı (S) |

---

## Özet Adımlar

```
1. FIBF  → BTE 1120'yi Z FM ile kayıt altına al
2. SE11  → BSEG_SUBST için ZFI_000_BSEG_SUBST append structure oluştur ve aktive et
3. SE37  → Z_FI_BTE_1120 function module oluştur, kodu yaz ve aktive et
4. FB01  → Test belgesi oluşturarak ZUONR alanının dolduğunu doğrula
```

---

*Sonraki bölüm: AC_DOCUMENT BADİ Uygulama Adımları*

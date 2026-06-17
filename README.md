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

# AC_DOCUMENT BADİ – Uygulama Adımları Rehberi

> **Kapsam:** FI Belge Posting – AC_DOCUMENT BADİ Implementasyonu  
> **İlgili Nesne:** BADİ: AC_DOCUMENT / Implementation: ZAC_DOCUMENT

---

## Adım 1 – SE18'de AC_DOCUMENT BADİ'sini Zle

1. Transaction: **SE18** aç.
2. **BADİ Definition** alanına `AC_DOCUMENT` yaz → **Display** ile aç.
3. Menüden **Implementation → Create** seçeneğine tıkla.
4. Aşağıdaki değerleri gir:

| Alan | Değer |
|---|---|
| Implementation Name | `ZAC_DOCUMENT` |
| Implementation Short Text | `ZAC_DOCUMENT` |
| Definition Name | `AC_DOCUMENT` |

5. **Interface** sekmesine geç. Aşağıdaki değerlerin otomatik geldiğini doğrula:

| Alan | Değer |
|---|---|
| Interface Name | `IF_EX_AC_DOCUMENT` |
| Name of Implementing Class | `ZCL_IM_AC_DOCUMENT` |

6. **Kaydet** ve **Aktive et**.

> ✅ Aktivasyon sonrası **Runtime Behavior** alanında `Implementation will be called` yazması gerekir.

---

## Adım 2 – ZCL_IM_AC_DOCUMENT Sınıfına Kodu Yaz

BADİ implementasyonu aktive edildikten sonra ilgili metodun içine kod yazılacaktır.

1. **Interface** sekmesinde metodun üzerine çift tıkla.
2. Hangi metoda kod yazılacağını belirle:

| Method | Açıklama |
|---|---|
| `CHANGE_INITIAL` | AC bileşenleri çağrılmadan önce FI/CO belgesi üzerinde değişiklik yapılır |
| `CHANGE_AFTER_CHECK` | AC bileşenleri eklendikten sonra FI/CO belgesi üzerinde değişiklik yapılır |
| `IS_SUPPRESSED_ACCT` | ACCT* tablolarının güncellenmemesi için kullanılır (SAP Note 48009) |
| `IS_COMPRESSION_REQUIRED` | İç tablolarda TABLE_COMPRESS uygulanır (SAP Note 320959) |
| `IS_ACCTIT_RELEVANT` | Ek AWTYPs için ACCT* güncellemesi aktive edilir |

3. `CHANGE_INITIAL` metodunu seç (çift tıkla) ve aşağıdaki kodu yaz:

```abap
DATA: lv_belnr TYPE bseg-belnr,
      lv_buzei TYPE bseg-buzei,
      lv_gjahr TYPE bseg-gjahr.

MOVE-CORRESPONDING im_document-header TO ex_document-header.

IF sy-tcode EQ 'FAGL_FCV'.

  LOOP AT im_document-item REFERENCE INTO DATA(lr_item).
    APPEND INITIAL LINE TO ex_document-item REFERENCE INTO DATA(lr_ex_item).
    MOVE-CORRESPONDING lr_item->* TO lr_ex_item->*.

    TRY.
        SPLIT lr_item->sgtxt AT ' ' INTO lv_belnr lv_buzei lv_gjahr DATA(lv_bos).

        lv_belnr = |{ lv_belnr ALPHA = IN }|.
        lv_buzei = |{ lv_buzei ALPHA = IN }|.

        SELECT SINGLE * FROM bseg
          WHERE bukrs EQ @lr_item->bukrs
            AND belnr EQ @lv_belnr
            AND buzei EQ @lv_buzei
            AND gjahr EQ @lv_gjahr
          INTO @DATA(ls_bseg).

        IF sy-subrc IS INITIAL.
          CASE ls_bseg-koart.
            WHEN 'K'.
              lr_ex_item->zuonr = |K-{ ls_bseg-lifnr }|.
            WHEN 'D'.
              lr_ex_item->zuonr = |D-{ ls_bseg-kunnr }|.
            WHEN 'S'.
              lr_ex_item->zuonr = |S-{ ls_bseg-hkont }|.
          ENDCASE.
        ENDIF.

      CATCH cx_root.
        MESSAGE e001(00) WITH 'Hatalı tanım!'.
    ENDTRY.

  ENDLOOP.

ENDIF.
```

> 💡 **Kodun mantığı:**
> - Yalnızca **`FAGL_FCV`** (Döviz Yeniden Değerleme) transaction'ında çalışır.
> - Her belge kalemi için `sgtxt` (açıklama) alanı boşlukla ayrılarak `belnr`, `buzei`, `gjahr` bilgilerine parse edilir.
> - Bu bilgilerle `BSEG` tablosundan orijinal belge satırı okunur.
> - Bulunan satırın `koart` değerine göre `zuonr` alanı `K-`, `D-` veya `S-` prefiksiyle doldurulur.
> - Hata durumunda `cx_root` yakalanarak kullanıcıya hata mesajı gösterilir.

4. **Kaydet** ve **Aktive et**.

---

## Adım 3 – Test Et

1. Transaction: **FAGL_FCV** (Döviz Yeniden Değerleme) aç.
2. İlgili parametreleri girerek çalıştır.
3. Oluşan belgeleri **FB03** ile aç → ilgili kalemlerin **ZUONR** alanının `K-`, `D-` veya `S-` prefiksiyle dolduğunu doğrula.
4. BADİ metodunun tetiklendiğini doğrulamak için gerekirse `CHANGE_INITIAL` metoduna **breakpoint** koy.

---

## Özet Adımlar

```
1. SE18  → AC_DOCUMENT BADİ'sini aç → ZAC_DOCUMENT implementasyonu oluştur ve aktive et
2. SE24  → ZCL_IM_AC_DOCUMENT sınıfı içinde ilgili metoda kodu yaz ve aktive et
3. FB01  → Test belgesi oluşturarak BADİ'nin tetiklendiğini doğrula
```

---

*Doküman sonu*

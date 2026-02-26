# 🏭 SAP ABAP – Ürün Ağacı & Stok Bazlı Üretilebilirlik Analizi

## 📋 Proje Özeti

Bu proje, **SAP ABAP** ortamında geliştirilmiş bir **Ürün Ağacı (BOM – Bill of Materials)** ve **Stok Yönetimi** uygulamasıdır. Program, ana malzemelerin alt malzeme bileşenlerine göre **maksimum üretilebilir miktarını** dinamik olarak hesaplar ve sonuçları **ALV Grid** üzerinde görselleştirir.

---

## 🎯 Projenin Amacı

Üretim planlamasında en kritik sorulardan biri şudur:

> *"Elimdeki stok miktarlarıyla, hangi ana malzemeden en fazla kaç adet üretebilirim?"*

Bu program, ürün ağacı yapısındaki alt malzeme gereksinimlerini ve mevcut stok durumunu analiz ederek bu soruya otomatik olarak yanıt verir.

---

## ⚙️ Teknik Detaylar

| Özellik | Detay |
|---|---|
| **Platform** | SAP ERP (ABAP) |
| **Dil** | ABAP (Object-Oriented) |
| **Mimari** | OOP – `lcl_class` sınıfı ile modüler yapı |
| **Çıktı** | Dinamik ALV Grid Raporu |
| **Veritabanı** | Z Tabloları (Custom Tables) |

### 📊 Veritabanı Tabloları

#### `Z182_UAGACI_T` – Ürün Ağacı Tablosu
Hangi ana malzemenin, hangi alt malzemelerden ve ne miktarda oluştuğunu tanımlar.

| Alan | Tip | Açıklama |
|---|---|---|
| `MANDT` | CLNT | Client |
| `ANA_MALZEME` | INT4 | Ana malzeme numarası |
| `ALT_MALZEME` | INT4 | Alt malzeme numarası |
| `ALT_MALZEME_MIKTARI` | INT4 | Gereken alt malzeme miktarı |

#### `Z182_STOK_T` – Stok Tablosu
Her alt malzemenin mevcut stok miktarını tutar.

| Alan | Tip | Açıklama |
|---|---|---|
| `MANDT` | CLNT | Client |
| `ALT_MALZEME` | INT4 | Alt malzeme numarası |
| `STOK_MIKTARI` | INT4 | Mevcut stok miktarı |

---

## 🧠 Hesaplama Mantığı

Program aşağıdaki algoritmayı uygular:

1. **Ürün ağacı verileri** (`Z182_UAGACI_T`) ve **stok verileri** (`Z182_STOK_T`) veritabanından çekilir.
2. Kayıtlar **ana malzemeye göre gruplanır** (`GROUP BY ana_malzeme`).
3. Her ana malzeme için tüm alt malzemeler döngüye alınır:
   - `Üretilebilir Miktar = Stok Miktarı / Gereken Alt Malzeme Miktarı` (tam bölüm – `FLOOR`)
4. Tüm alt malzemeler arasındaki **minimum değer**, o ana malzemenin **üretilebilir miktarı** olarak belirlenir.
   - *Darboğaz prensibi: En az stoku olan bileşen, üretimi sınırlar.*
5. Eğer herhangi bir alt malzemenin stoğu yoksa, üretilebilir miktar **0** olur.

### 📌 Örnek Hesaplama

| Ana Malzeme | Alt Malzeme | Gereken Miktar | Stok | Üretilebilir |
|---|---|---|---|---|
| 1.000 | 9.000 | 5 | 100 | 100/5 = **20** |
| 1.000 | 9.001 | 10 | 100 | 100/10 = **10** |
| 1.000 | 9.002 | 15 | 100 | 100/15 = **6** |
| **Sonuç** | | | | **min(20,10,6) = 6** → *(6+1=7 ekran görüntüsündeki fark, sınır malzeme tam bölünme)* |

---

## 🖥️ Ekran Görüntüleri

### ALV Rapor Çıktısı
Dinamik ALV Grid üzerinde ana malzemelerin üretilebilir miktarları:

![ALV Rapor Çıktısı](Ekran%20görüntüsü%202026-02-26%20230327.png)

### Ürün Ağacı Tablosu – `Z182_UAGACI_T`

Tablo yapısı (ABAP Dictionary):

![Ürün Ağacı Tablo Yapısı](Ekran%20görüntüsü%202026-02-26%20230353.png)

Tablo verileri:

![Ürün Ağacı Verileri](Ekran%20görüntüsü%202026-02-26%20230456.png)

### Stok Tablosu – `Z182_STOK_T`

Tablo yapısı (ABAP Dictionary):

![Stok Tablo Yapısı](Ekran%20görüntüsü%202026-02-26%20230408.png)

Tablo verileri:

![Stok Verileri](Ekran%20görüntüsü%202026-02-26%20230436.png)

---

## 🔑 Öne Çıkan Teknik Özellikler

- **🧩 Dinamik ALV Yapısı**: Alt malzeme sayısına göre ALV kolonları otomatik olarak oluşturulur. Yeni bir alt malzeme eklendiğinde ALV otomatik olarak güncellenir – sabit kolon tanımlamasına gerek yoktur.
- **🏗️ Nesne Yönelimli Mimari (OOP)**: Tüm iş mantığı `lcl_class` sınıfı içinde metotlara ayrılmıştır (`get_data`, `set_fcat`, `set_layout`, `display_alv`, `set_alv_structure`).
- **⚡ Dinamik Internal Table**: `cl_alv_table_create=>create_dynamic_table` ile çalışma anında (runtime) tablo yapısı oluşturulur.
- **🔄 Field-Symbols & ASSIGN COMPONENT**: Dinamik kolon erişimi için ABAP'ın en güçlü araçları kullanılır.
- **📐 GROUP BY Döngü**: ABAP 7.40+ ile gelen modern `LOOP AT ... GROUP BY` sözdizimi kullanılmıştır.

---

## 🛠️ Kullanılan Teknolojiler

`SAP ERP` · `ABAP OOP` · `ALV Grid` · `ABAP Dictionary` · `Field-Symbols` · `Dynamic Programming` · `Screen Programming (Dynpro)`

---

## 📂 Proje Yapısı

```
z_182_uruntree_mlzm
├── z_182_uruntree_mlzm_top   → Global veri tanımlamaları (TOP Include)
├── z_182_uruntree_mlzm_cls   → Sınıf tanımı ve implementasyonu
└── z_182_uruntree_mlzm_mdl   → Modül havuzu (PBO/PAI)
```

---

## 👨‍💻 Geliştirici

Bu proje, SAP ABAP eğitimi kapsamında geliştirilmiştir.

---

> **#SAP #ABAP #ALV #BillOfMaterials #ÜrünAğacı #StokYönetimi #DynamicALV #OOP #SAPDevelopment #ERPDevelopment**


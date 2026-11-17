# Klasik Görüntü İşleme ile Hücre Segmentasyonu ve Sayımı (Fluo-N2DH-GOWT1)

Cell Tracking Challenge **Fluo-N2DH-GOWT1** eğitim verisinde klasik görüntü işleme yöntemleriyle
hücre segmentasyonu, sayımı ve zamansal sayım analizi. Derin öğrenme kullanılmamıştır.

## Veri seti
- Resmî indirme bağlantısı: https://data.celltrackingchallenge.net/training-datasets/Fluo-N2DH-GOWT1.zip
- Notebook arşivi otomatik olarak indirip `data/` klasörüne açar. **Veri seti bu repoda yer almaz.**

## Tekrarlanabilirlik
- Random seed: **14048** (öğrenci numarası son 5 hane 12022 + 2026, aritmetik toplam)
- Zamansal boşluk kuralı: iki kare arası fark > 5
- Seçilen kareler: `selected_frames.csv`

## Kurulum ve çalıştırma
1. Google Colab'da `main.ipynb` dosyasını açın (yerelde: `pip install -r requirements.txt`).
2. *Çalışma zamanı → Tümünü çalıştır.* Veri seti indirilir, tüm deneyler, tablolar ve şekiller üretilir.
3. Bölümler sırasıyla çalışır: veri inceleme → seçim → referanslar → deneyler → parametre taramaları →
   doğrulama → **kilitleme** → test değerlendirmesi → zamansal analiz → şekiller → hata analizi.

## Nihai pipeline
**Pipeline B (CLAHE+Gauss5+Adaptif+CC)** (`E11`): Girdi görüntüsü (1024×1024, uint8) → CLAHE clip=3.0, grid=[8, 8] → Gauss filtre 5×5 → Adaptif Gauss eşik blok=71, C=-12 → Bağlı bileşen + alan filtresi (≥ 100 px) → sayım

Seçim, en düşük doğrulama MAE değerine göre otomatik yapılmıştır (`validation_results.csv`).
Parametreler `final_pipeline.json` dosyasındadır.

| Yöntem | Test MAE | Test RMSE | Bias | Exact (%) |
|---|---|---|---|---|
| Baseline (E01) | 13.067 | 13.99 | -13.067 | 6.7 |
| Nihai (E11) | 10.667 | 10.801 | -10.667 | 0.0 |

## Test güvenliği
Nihai pipeline ve test tahminleri (`final_pipeline.json`, `test_predictions_locked.csv`), test referansları
okunmadan ayrı bir commit ile kaydedilmiştir. Test değerlendirmesi sonraki commit'tedir.
Kod, sekans 02 referanslarının kilit dosyası oluşmadan okunmasını engeller.

## Üretilen dosyalar
| Dosya | İçerik |
|---|---|
| `data_overview.csv` | Tüm karelerin boyut, tip ve yoğunluk istatistikleri |
| `selected_frames.csv` | Seçilen kareler, gruplar, test blokları ve seed (referans sayısı içermez) |
| `dev_val_references.csv` | Geliştirme ve doğrulama referans hücre sayıları |
| `experiments.csv` / `experiments_per_frame.csv` | 20 kontrollü deney (Tablo 2) ve kare bazlı tahminler |
| `parameter_sweeps.csv` | Parametre duyarlılık analizi (geliştirme kümesi) |
| `validation_results.csv` | Üç aday pipeline doğrulama sonuçları (Tablo 3) |
| `final_pipeline.json` | Nihai pipeline ve baseline konfigürasyonu |
| `test_predictions_locked.csv` | Referanslar okunmadan kaydedilen test tahminleri |
| `test_results.csv` / `test_summary.csv` | Test sonuçları (Tablo 4) ve özet metrikler (Tablo 5) |
| `validation_vs_test.csv` | Doğrulama ve test karşılaştırması |
| `temporal_analysis.csv` | Ardışık karelerdeki sayım değişimleri |
| `error_analysis_*.csv` | Hücre ve kare bazında hata ayrıştırması |
| `figures/` | Veri seti görüntüsü içermeyen grafikler |

Veri seti görüntüsü içeren şekiller (`figures_report/`) yalnızca teknik raporda yer alır ve repoya eklenmez.

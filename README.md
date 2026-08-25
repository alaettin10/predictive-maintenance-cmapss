# NASA C-MAPSS ile Predictive Maintenance

## Proje Hakkında

Bu projede NASA C-MAPSS FD001 veri seti kullanılarak uçak motorlarının **Remaining Useful Life (RUL)** değerlerinin tahmin edilmesi amaçlanmıştır.

Amaç, motorların çalışma döngüleri boyunca sensör verilerindeki degradation (yıpranma) eğilimlerini kullanarak motorun kalan faydalı ömrünü tahmin eden bir makine öğrenmesi modeli geliştirmektir.

Proje kapsamında veri analizi, feature engineering, feature selection, model geliştirme, GroupKFold validation, test değerlendirmesi ve hata analizi gerçekleştirilmiştir.

---

## Veri Seti

Projede **NASA C-MAPSS FD001** veri seti kullanılmıştır.

Veri setinde uçak motorlarının farklı çalışma döngülerindeki sensör ölçümleri bulunmaktadır.

Temel kolonlar:

* `unit_number`: Motor kimliği
* `time_in_cycles`: Motorun çalışma döngüsü
* `op_setting_1`, `op_setting_2`, `op_setting_3`: Operasyon ayarları
* `sensor_1` - `sensor_21`: Sensör ölçümleri

FD001 veri setinde toplam **100 motor** bulunmaktadır.

Tahmin edilmeye çalışılan değer **Remaining Useful Life (RUL)** değeridir.

---

## Proje Akışı

Projede aşağıdaki aşamalar izlenmiştir:

```text
Ham Veri
   ↓
EDA
   ↓
Sabit Sensörlerin Çıkarılması
   ↓
Korelasyon Analizi
   ↓
RUL Target Hazırlama
   ↓
Feature Engineering
   ↓
Feature Selection
   ↓
XGBoost
   ↓
GroupKFold Validation
   ↓
Final Model
   ↓
Test Değerlendirmesi
   ↓
Hata Analizi
```

---

## Exploratory Data Analysis

İlk olarak sensörlerin dağılımları, zaman içerisindeki değişimleri ve RUL ile ilişkileri incelenmiştir.

EDA sırasında özellikle:

* Sabit veya neredeyse sabit sensörler
* Sensörlerin zaman içerisindeki trendleri
* Sensörler arasındaki korelasyonlar
* Sensörler ile RUL arasındaki korelasyonlar
* RUL dağılımı

incelenmiştir.

### Sabit Sensörlerin Çıkarılması

Varyansı yaklaşık sıfır olan 7 sensör belirlenmiştir:

```text
sensor_1
sensor_5
sensor_6
sensor_10
sensor_16
sensor_18
sensor_19
```

Bu sensörler FD001 veri setinde pratik olarak anlamlı bilgi taşımadıkları için çıkarılmıştır.

Bu karar **ADR-001** içerisinde dokümante edilmiştir.

---

## Feature Engineering

Sensör verilerindeki kısa süreli dalgalanmaları azaltmak ve zaman içerisindeki davranışları daha iyi temsil etmek amacıyla 10 cycle'lık rolling window kullanılmıştır.

Oluşturulan feature türleri:

* Rolling Mean
* Rolling Standard Deviation
* Rolling Trend

Örneğin `sensor_11` için:

```text
sensor_11
sensor_11_rolling_mean_10
sensor_11_trend_10
```

feature'ları oluşturulmuştur.

Motor 1 üzerinde yapılan incelemede:

```text
sensor_11                  → -0.842
sensor_11_rolling_mean_10  → -0.901
sensor_11_trend_10         → -0.618
```

korelasyonları gözlemlenmiştir.

Rolling mean'in RUL ile daha güçlü negatif korelasyona sahip olduğu görülmüştür.

Ancak bu sonuç yalnızca tek motor üzerinden elde edildiği için rolling mean'in kesin olarak daha iyi olduğu yalnızca bu sonuç üzerinden kabul edilmemiş, model validation sonuçlarıyla birlikte değerlendirilmiştir.

### Rolling Standard Deviation

Rolling standard deviation tüm sensörlerde denenmiştir.

En yüksek mutlak RUL korelasyonu:

```text
sensor_9_rolling_std_10 → -0.227
```

olarak bulunmuştur.

Bu nedenle rolling standard deviation'ın RUL ile doğrudan doğrusal ilişkisinin genel olarak zayıf olduğu görülmüştür.

---

## Feature Selection

Feature selection aşamasında korelasyon analizi ve model tabanlı feature importance sonuçları kullanılmıştır.

### Sensor 14'ün Çıkarılması

`Sensor 9` ve `Sensor 14` arasında:

```text
Correlation = 0.963
```

bulunmuştur.

Bu iki sensörün RUL ile korelasyonları:

```text
sensor_9  → -0.390
sensor_14 → -0.307
```

olarak bulunmuştur.

Mutlak korelasyon açısından `sensor_9` RUL ile daha güçlü ilişkiye sahip olduğu için `sensor_9` tutulmuş ve `sensor_14` çıkarılmıştır.

Bu karar **ADR-002** içerisinde dokümante edilmiştir.

### RUL Target Clipping

Gerçek RUL değerleri değiştirilmemiştir.

Model eğitiminde kullanılmak üzere ayrı bir `RUL_target` değişkeni oluşturulmuştur.

Kural:

```text
RUL > 125  → RUL_target = 125
RUL ≤ 125 → RUL_target = RUL
```

Bunun amacı motorun erken yaşam dönemlerinde degradation bilgisinin daha belirsiz olduğu bölgenin etkisini azaltarak modelin daha anlamlı degradation bölgesine odaklanmasını sağlamaktır.

Gerçek `RUL` değerleri korunmuştur.

Bu karar **ADR-003** içerisinde dokümante edilmiştir.

---

## Model Development

Model geliştirme aşamasında **XGBoost Regressor** kullanılmıştır.

Farklı hyperparameter değerleri denenmiştir:

* `learning_rate`
* `n_estimators`
* `max_depth`
* `subsample`
* `colsample_bytree`

Final modelde kullanılan parametreler:

```text
n_estimators = 200
max_depth = 3
learning_rate = 0.03
subsample = 1.0
colsample_bytree = 0.6
random_state = 42
objective = reg:squarederror
```

---

## Validation Strategy

Model değerlendirmesinde **GroupKFold** kullanılmıştır.

Gruplama değişkeni:

```text
unit_number
```

olarak belirlenmiştir.

Bunun temel nedeni aynı motora ait farklı cycle'ların hem training hem validation setinde bulunmasını engellemektir.

Bu yaklaşım aynı motorun farklı cycle'larından kaynaklanabilecek data leakage riskini azaltmaktadır.

Model performansı:

* MAE
* RMSE

metrikleri kullanılarak değerlendirilmiştir.

---

## Final Model

Final model **XGBoost Regressor** olarak belirlenmiştir.

Model:

```text
n_estimators = 200
max_depth = 3
learning_rate = 0.03
subsample = 1.0
colsample_bytree = 0.6
random_state = 42
```

parametreleriyle eğitilmiştir.

Feature engineering ve feature selection aşamalarından sonra oluşturulan final feature seti kullanılmıştır.

---

## Results

Final test sonuçları:

| Metrik |     Sonuç |
| ------ | --------: |
| MAE    | **33.80** |
| RMSE   | **45.11** |

**MAE**, tahminlerin gerçek RUL değerlerinden ortalama mutlak sapmasını göstermektedir.

**RMSE**, büyük hataları MAE'ye göre daha fazla cezalandırdığı için modelin büyük tahmin hatalarına karşı davranışını değerlendirmede kullanılmıştır.

---

## Error Analysis

Test tahminlerinden sonra her gözlem için:

* Actual RUL
* Predicted RUL
* Error
* Absolute Error

değerleri hesaplanmıştır.

Hata analizi kapsamında:

### Actual vs Predicted RUL

Gerçek ve tahmin edilen RUL değerleri scatter plot ile karşılaştırılmıştır.

Grafikte `y = x` çizgisi ideal tahmin durumunu göstermektedir.

### Prediction Error Distribution

Tahmin hatalarının dağılımını incelemek amacıyla histogram oluşturulmuştur.

Ayrıca motor bazında ortalama ve maksimum mutlak hatalar incelenerek modelin hangi motorlarda daha fazla hata yaptığı araştırılmıştır.

---

## Project Structure

```text
predictive-maintenance-cmapss/
│
├── 01_data_exploration.ipynb
│
├── train_FD001.txt
├── test_FD001.txt
├── RUL_FD001.txt
│
├── ADR-001-remove-constant-sensors.md
├── ADR-002-remove-sensor-14.md
├── ADR-003-rul-target-clipping.md
├── ADR-004-use-of-rolling-mean-and-trend-features.md
├── ADR-005-evaluation-of-rolling-standard-deviation.md
│
├── README.md
├── requirements.txt
└── .gitignore
```

---

## Installation

Projeyi klonladıktan sonra Python virtual environment oluşturulabilir:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Gerekli Python paketleri:

```bash
pip install -r requirements.txt
```

---

## Usage

Projeyi çalıştırmak için:

```text
01_data_exploration.ipynb
```

notebook'u açılır ve hücreler sırasıyla çalıştırılır.

Notebook içerisinde:

* Veri yükleme
* Veri analizi
* Feature engineering
* Feature selection
* Model geliştirme
* Validation
* Test tahmini
* Hata analizi

aşamaları gerçekleştirilmektedir.

---

## ADRs

Projede alınan önemli teknik kararlar Architecture Decision Record (ADR) dosyaları ile dokümante edilmiştir.

* [ADR-001: Remove Constant Sensors](ADR-001-remove-constant-sensors.md)
* [ADR-002: Remove Sensor 14](ADR-002-remove-sensor-14.md)
* [ADR-003: RUL Target Clipping](ADR-003-rul-target-clipping.md)
* [ADR-004: Use of Rolling Mean and Trend Features](ADR-004-use-of-rolling-mean-and-trend-features.md)
* [ADR-005: Evaluation of Rolling Standard Deviation](ADR-005-evaluation-of-rolling-standard-deviation.md)

---

## Future Improvements

Projede ileride değerlendirilebilecek geliştirmeler:

* Farklı regression modellerinin karşılaştırılması
* Daha kapsamlı hyperparameter tuning
* Yeni time-series feature'larının denenmesi
* Feature selection yöntemlerinin geliştirilmesi
* Motor bazlı hata analizinin genişletilmesi
* RUL target yaklaşımının farklı sınır değerleriyle karşılaştırılması
* Model performansının farklı C-MAPSS veri setlerinde değerlendirilmesi
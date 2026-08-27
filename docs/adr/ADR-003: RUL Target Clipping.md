# ADR-003: RUL Target Clipping

## Problem

Öncelikle RUL dağılımını bir histogram ile inceledik. RUL değerlerinin düşük değerlerde daha yoğun olduğunu ve RUL arttıkça örnek sayısının azaldığını gözlemledik.

Daha sonra sensörlerin RUL ile olan ilişkilerini görsel olarak inceledik. Örnek olarak RUL ile güçlü negatif korelasyona sahip olan `sensor_11` incelendi. Grafik dalgalı olmasına rağmen RUL azaldıkça `sensor_11` değerinin genel olarak arttığı görüldü.

Diğer sensörlerde de benzer dalgalanmalar gözlemlendi. Ancak genel trendlerin sensöre göre değiştiği görüldü. Negatif korelasyona sahip sensörlerde RUL azalırken sensör değerinin genel olarak arttığı, pozitif korelasyona sahip sensörlerde ise bunun tersi yönde bir eğilim olduğu gözlemlendi.

Bu incelemeler sonucunda motorların erken yaşam dönemlerinde sensör değişimlerinin daha belirsiz ve dalgalı olabileceği değerlendirildi.

## Decision

Gerçek RUL değerini koruyarak, model eğitiminde kullanılmak üzere ayrı bir `RUL_target` değişkeni oluşturulmasına karar verildi.

`RUL_target` için 125 cycle üst sınırı kullanıldı:

* `RUL > 125` → `RUL_target = 125`
* `RUL ≤ 125` → `RUL_target = RUL`

Gerçek `RUL` sütunu değiştirilmedi.

## Why

Amaç, motorların erken yaşam dönemlerinde henüz belirgin bir degradation göstermeyen bölgelerdeki yüksek RUL değerlerinin model üzerindeki etkisini azaltmak ve modelin daha anlamlı degradation bölgesine odaklanmasını sağlamaktır.

Bu yaklaşım ile modelin 125'in üzerindeki RUL değerlerini birbirinden ayrı tahmin etmek yerine bu değerleri aynı üst seviyede değerlendirmesi hedeflenmiştir.

## Trade-offs

RUL değerlerinin 125 cycle'dan sonra tek bir üst seviyede temsil edilmesi, modelin erken yaşam dönemlerindeki farklı RUL değerlerini birbirinden ayırt etme yeteneğini azaltabilir.

Ancak gerçek `RUL` değerinin ayrı tutulması sayesinde bu bilgi veri setinden tamamen kaybedilmemektedir.

## Risks

125 cycle sınırı veri analizi ve modelleme yaklaşımına dayalı bir seçimdir. Farklı veri setlerinde veya farklı problem tanımlarında uygun üst sınır değişebilir.

Bu nedenle clipping sınırının farklı veri setlerinde yeniden değerlendirilmesi gerekir.

## Evaluation

Model eğitiminde `RUL_target` kullanılırken, test aşamasında gerçek `RUL` değerleri korunarak model performansının gerçek RUL'a karşı değerlendirilmesi hedeflenmiştir.

Gerçek `RUL` değerinin korunmasının bir diğer nedeni, ileride farklı target yaklaşımlarının ve model performanslarının karşılaştırılabilmesidir.
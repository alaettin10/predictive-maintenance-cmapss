# ADR-001: Sabit Sensörlerin Çıkarılması

## Problem

FD001 veri setindeki 21 sensör incelendiğinde `sensor_1`, `sensor_5`, `sensor_6`, `sensor_10`, `sensor_16`, `sensor_18` ve `sensor_19` sensörlerinin varyanslarının yaklaşık sıfır olduğu görüldü.

Bu sensörler veri boyunca sabit veya neredeyse sabit değerler taşıdığı için model açısından anlamlı ayırt edici bilgi sağlamamaktadır.

## Decision

Bu 7 sensörün hem eğitim hem de test veri setinden çıkarılmasına karar verildi.

Çıkarılan sensörler:

* `sensor_1`
* `sensor_5`
* `sensor_6`
* `sensor_10`
* `sensor_16`
* `sensor_18`
* `sensor_19`

## Why

Sabit veya neredeyse sabit feature'lar modelin tahmin performansına anlamlı katkı sağlamaz. Bu feature'ları çıkarmak:

* Gereksiz feature sayısını azaltır.
* Veri setini sadeleştirir.
* Modelin gereksiz feature'ları değerlendirmesini önler.
* Hesaplama maliyetini küçük de olsa azaltır.

## Trade-offs

Bu sensörlerin FD001 içerisinde anlamlı bilgi taşımadığı gözlemlendiği için bilgi kaybı riski düşüktür. Ancak farklı bir veri setinde aynı sensörler değişkenlik gösterebilir ve faydalı bilgi taşıyabilir.

## Risks

Bu karar yalnızca **C-MAPSS FD001** veri setindeki gözlemlere dayanmaktadır.

FD002, FD003 veya FD004 gibi farklı çalışma koşullarına sahip veri setlerinde bu sensörlerin sabit olduğu varsayılmamalıdır. Yeni bir veri seti kullanıldığında sensör varyansları yeniden analiz edilmelidir.
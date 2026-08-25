## ADR-002: Use of Rolling Mean and Trend Features


## Problem

Sensör verilerinde kısa süreli dalgalanmalar ve gürültü bulunabilmektedir. Bu dalgalanmalar, sensörün zaman içerisindeki genel davranışını doğrudan gözlemlemeyi zorlaştırabilir.

Bu nedenle sensör verilerinden zaman serisi tabanlı yeni feature'lar oluşturulması değerlendirildi.

## Decision

`Sensor_11` üzerinde 10 cycle'lık rolling mean ve rolling mean üzerinden 10 cycle'lık trend feature'ı oluşturulmasına karar verildi.

Rolling mean hesaplanırken `min_periods=1` kullanıldı. Böylece ilk 9 cycle'ın feature üretimi sırasında tamamen kaybedilmesi önlendi.

Oluşturulan feature'lar:

* `sensor_11_rolling_mean_10`
* `sensor_11_trend_10`

## Why

Rolling mean'in amacı sensördeki kısa süreli dalgalanmaları azaltarak daha genel bir sinyal elde etmektir.

Trend feature'ının amacı ise sensör değerinin zaman içerisinde hangi yönde değiştiğini temsil etmektir.

İlk incelemede Motor 1 üzerinde aşağıdaki RUL korelasyonları elde edildi:

| Feature                     | RUL korelasyonu |
| --------------------------- | --------------: |
| `sensor_11`                 |          -0.842 |
| `sensor_11_rolling_mean_10` |          -0.901 |
| `sensor_11_trend_10`        |          -0.618 |

Rolling mean'in ham `sensor_11` değerine göre RUL ile daha güçlü negatif korelasyona sahip olduğu görüldü.

## Trade-offs

Rolling mean kısa süreli dalgalanmaları azaltırken hızlı değişimleri de yumuşatabilir. Ayrıca trend feature'ı oluşturulurken pencerenin başındaki cycle'larda yeterli geçmiş veri bulunmayabilir.

10 cycle'lık pencere, sinyali aşırı yumuşatmadan kısa süreli dalgalanmaları azaltmak amacıyla seçildi.

## Validation

Motor 1 üzerindeki korelasyon sonuçları rolling mean'in potansiyel olarak faydalı olduğunu gösterse de bu sonuç tek bir motor üzerinden elde edilmiştir.

Bu nedenle rolling mean'in kesin olarak daha iyi bir feature olduğu yalnızca bu korelasyon sonucuna dayanarak kabul edilmemiştir. Feature'ın faydası tüm motorlar ve model validation sonuçları üzerinden değerlendirilmiştir.
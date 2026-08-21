# ADR-003: RUL Target Clipping
Öncelikle RUL dağılımını bir histogram ile inceledik. Grafikte RUL değerlerinin düşük değerlerde daha yoğun olduğunu ve RUL arttıkça örnek sayısının azaldığını gözlemledik.

Daha sonra sensörlerin RUL ile olan ilişkilerini görsel olarak inceledik. Örnek olarak RUL ile güçlü negatif korelasyona sahip olan Sensor 11'e baktık. Grafik dalgalı olmasına rağmen RUL azaldıkça Sensor 11'in genel olarak arttığını gördük.

Diğer sensörleri de incelediğimizde benzer şekilde dalgalanmalar olduğunu, ancak genel trendlerin sensöre göre değiştiğini gördük. Negatif korelasyona sahip sensörlerde RUL azalırken sensör değeri genel olarak artarken, pozitif korelasyona sahip sensörlerde bunun tersi yönde bir eğilim gözlemledik.

Bu incelemeler sonucunda motorun erken yaşam dönemlerinde sensörlerdeki değişimlerin daha belirsiz ve dalgalı olabileceğini değerlendirdik.
Gerçek RUL değerini koruyarak, modelin eğitiminde kullanılmak üzere ayrı bir RUL_target değişkeni oluşturulmasına karar verildi.
RUL_target için 125 cycle üst sınırı kullanıldı.
Örneğin:

- RUL > 125 → RUL_target = 125
- RUL ≤ 125 → RUL_target = RUL

Gerçek RUL sütunu değiştirilmedi.
Amaç, motorun erken ve henüz belirgin degradation göstermeyen dönemlerinde modelin gereğinden fazla belirsiz bilgi öğrenmesini azaltmak ve daha anlamlı degradation bölgesine odaklanmasını sağlamaktır.

Gerçek RUL değerini korumamızın nedeni ise ileride farklı target yaklaşımlarını ve model performanslarını karşılaştırabilmektir.

- Model 125'in üzerindeki RUL değerlerini ayrı ayrı tahmin etmek yerine bunları aynı üst seviyede değerlendirecek.
- Erken yaşam dönemindeki gereksiz ayrıntıların etkisinin azaltılması hedefleniyor.
- Gerçek RUL bilgisi kaybedilmedi.
- RUL_target ve gerçek RUL ileride model performansını değerlendirmek için ayrı ayrı kullanılabilir.
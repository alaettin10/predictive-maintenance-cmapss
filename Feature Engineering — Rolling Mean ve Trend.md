Feature engineering aşamasında öncelikle sensor_11 üzerinden 10 cycle'lık rolling mean oluşturduk. Amacımız sensördeki kısa süreli dalgalanmaları azaltarak daha genel bir sinyal elde etmekti. İlk 9 cycle'ı kaybetmemek için min_periods=1 kullandık.

Daha sonra rolling mean üzerinden 10 cycle'lık trend oluşturduk. Trendin amacı sensörün zaman içerisinde hangi yönde değiştiğini görmekti. İlk cycle'da yeterli veri olmadığı için NaN oluştu.

Son olarak sensor_11, sensor_11_rolling_mean_10 ve sensor_11_trend_10 değerlerinin RUL ile korelasyonlarını karşılaştırdık. Motor 1 üzerinde sırasıyla -0.842, -0.901 ve -0.618 değerlerini elde ettik. Rolling mean'in RUL ile daha güçlü negatif korelasyona sahip olduğunu gördük.

Bu sonuç doğrultusunda rolling mean'in sensörün kısa süreli dalgalanmalarını azaltarak RUL ile olan ilişkiyi daha belirgin hale getirebileceğini gördük. Ancak bu sonuç yalnızca tek motor üzerinden elde edildiği için rolling mean'in kesin olarak daha iyi bir feature olduğuna henüz karar vermedik. Bunun tüm motorlar ve model validation sonuçları üzerinden ayrıca değerlendirilmesine karar verdik.
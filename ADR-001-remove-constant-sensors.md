## ADR - Sabit Sensörlerin Çıkarılması
Problem: 21 sensörden 7 tanesi (sensor_1,5,6,10,16,18,19) neredeyse 
sıfır varyansa sahip (std ~0), yani pratikte hiç bilgi taşımıyor.
Decision: Çıkarmayı seçtim.
Why: Model gereksiz gürültüden etkilenmesin, hesaplama da hafifler.
Trade-offs: Yok denecek kadar az  bu sensörler zaten bilgi taşımıyor.
Risks: Farklı bir FD veri setinde (FD002-004) bu sensörler sabit 
olmayabilir, o yüzden bu kararı her veri setinde tekrar kontrol etmek gerekir.
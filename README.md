Kullanılan dataset: https://www.kaggle.com/datasets/asaniczka/amazon-products-dataset-2023-1-4m-products



---

# 2- Ürün Başlıklarından Kategori Tahmini: TF-IDF ve Lojistik Regresyon Yaklaşımı

Bu proje, Amazon ürün veri setini kullanarak, ürün başlıklarından (`title`) yola çıkıp ürünün ait olduğu kategoriyi (`category_name`) tahmin eden bir makine öğrenmesi modeli geliştirmeyi amaçlamaktadır. Proje, Keşifsel Veri Analizi (EDA), sınıf dengesizliği probleminin tespiti ve çözümü, metin ön işleme, TF-IDF ile özellik çıkarma ve Lojistik Regresyon ile sınıflandırma adımlarını kapsamaktadır.

## Proje Hedefi

E-ticaret platformlarında milyonlarca ürün bulunmaktadır ve bu ürünlerin doğru kategorilere ayrılması hem kullanıcı deneyimi hem de envanter yönetimi için kritiktir. Bu projenin temel hedefi, bu süreci otomatikleştirebilecek, bir ürün başlığı verildiğinde %90'ın üzerinde doğrulukla kategorisini tahmin edebilen bir model oluşturmaktır.

---

## 1. Problem Tespiti ve Strateji Geliştirme

Projenin ilk ve en önemli adımı, veri setinin yapısını anlamak ve potansiyel problemleri tespit etmekti.

### Problem: Aşırı Sınıf Dengesizliği (Class Imbalance)

Veri seti incelendiğinde, kategoriler arasında ürün sayısı bakımından büyük bir dengesizlik olduğu tespit edildi. Toplam 248 farklı kategori bulunurken:
*   En kalabalık kategorilerde (örn: `Girls' Clothing`) **28,000+** ürün,
*   En az ürüne sahip kategorilerde (örn: `Smart Home Thermostats`) ise sadece **22** ürün vardı.

Bu durum, makine öğrenmesi modelleri için ciddi bir sorundur. Model, veri setindeki baskın olan kategorileri ("çoğunluk sınıfları") ezberlemeye meyilli olur ve azınlıkta kalan kategorileri doğru tahmin etmekte zorlanır.

**Kategorilerdeki Ürün Sayılarının Dağılımı**
*Aşağıdaki histogram, veri setindeki kategorilerin büyük bir kısmının 10.000'den az ürüne sahip olduğunu ve çok az sayıda kategorinin aşırı derecede baskın olduğunu açıkça göstermektedir.*

![image](https://github.com/user-attachments/assets/03d6f47a-c27c-4e67-93a9-9dd13a32654b)


### Uygulanan Strateji: Kontrollü ve Dengeli Veri Seti Oluşturma

Bu dengesizlik problemini çözmek ve daha genellenebilir bir model eğitmek için aşağıdaki strateji izlenmiştir:

1.  **Hedef Kategori Havuzunu Belirleme:** Ne çok baskın ne de çok seyrek olan, yani "makul" sayıda ürüne sahip kategoriler hedeflendi. Bu amaçla, **1000 ile 2000 arasında ürüne sahip kategoriler** filtrelendi. Bu aralık, modelin öğrenmesi için yeterli veri sağlarken, aşırı baskınlık yaratmayacak bir denge sunar.

2.  **Temsili Kategorileri Seçme:** Bu filtrelenmiş havuzdan, çeşitli alanları temsil eden **10 adet kategori** manuel olarak seçildi.

3.  **Dengeli Alt Örneklem (Balanced Sub-sampling):** En kritik adım olarak, seçilen bu 10 kategorinin her birinden **rastgele 600'er adet örnek** alındı. Bu işlem, `groupby('category_name').sample(n=600)` metodu ile gerçekleştirildi.

Bu strateji sayesinde, her sınıfın eşit temsil edildiği, toplam **10 kategori ve 6000 satırdan oluşan mükemmel dengeli bir veri seti** oluşturuldu. Bu, modelin her kategoriye eşit derecede önem vermesini sağladı.

**Oluşturulan Dengeli Veri Seti**
*Her kategoriden tam olarak 600 örnek alındığı görülmektedir, bu da sınıf dengesizliği sorununu tamamen ortadan kaldırır.*

![image](https://github.com/user-attachments/assets/b3bc1960-4960-4db0-a83b-0efaab368a38)

---

## 2. Metodoloji ve İş Akışı

Dengeli veri seti oluşturulduktan sonra standart bir NLP ve sınıflandırma iş akışı takip edildi.

### Adım 1: Metin Ön İşleme

Modelin metinleri daha iyi anlaması için ürün başlıkları (`title`) bir dizi işlemden geçirildi:
*   **Küçük Harfe Çevirme:** Tüm metinler küçük harfe dönüştürüldü.
*   **Noktalama ve Sayıların Kaldırılması:** Modelin kafasını karıştırabilecek özel karakterler, sayılar ve noktalama işaretleri temizlendi.
*   **Gereksiz Kelimelerin (Stop Words) Çıkarılması:** "and", "the", "for" gibi anlamsal değeri olmayan yaygın kelimeler metinden ayıklandı.
*   **Kök Bulma (Lemmatization):** Kelimeler, anlamsal köklerine indirgendi (örn: "cards", "card's" -> "card"). Bu, kelime dağarcığını (vocabulary) küçülterek modelin performansını artırır.

**Ön İşleme Örneği:**
*   **Orijinal Başlık:** `Fox Baby Shower Thank You Cards Woodland Forest Friends...`
*   **Temizlenmiş Başlık:** `fox baby shower thank card woodland forest friend animal...`

### Adım 2: TF-IDF Vektörleştirme

Temizlenmiş metinler, makine öğrenmesi modelinin işleyebileceği sayısal formatlara dönüştürüldü. Bunun için **TF-IDF (Term Frequency-Inverse Document Frequency)** tekniği kullanıldı. TF-IDF, bir kelimenin belirli bir başlık için önemini, tüm başlıklardaki genel yaygınlığına göre tartan bir skorlama yöntemidir.

*   `max_features=5000`: Modelde en önemli 5000 kelime/terim kullanıldı.
*   `ngram_range=(1, 2)`: Model, hem tekli kelimeleri ("toy") hem de ikili kelime gruplarını ("race track") dikkate aldı. Bu, daha zengin bir anlamsal bağlam yakalanmasını sağlar.

### Adım 3: Model Eğitimi ve Değerlendirme

Veri seti, **%85 eğitim** ve **%15 test** verisi olarak ayrıldı. Ayırma işlemi sırasında `stratify=y` parametresi kullanılarak, test ve eğitim setlerinde kategori dağılımlarının dengeli kalması garanti altına alındı. Bu veri üzerinde **Lojistik Regresyon** modeli eğitildi.

**Model Performansı:**
Model, test seti üzerinde **%93.67 (yaklaşık %94) doğruluk** oranına ulaştı. Aşağıdaki sınıflandırma raporu, modelin her bir kategori için **precision (kesinlik)** ve **recall (duyarlılık)** metriklerinde ne kadar başarılı olduğunu detaylı olarak göstermektedir.

![image](https://github.com/user-attachments/assets/fd07ccf3-7ba4-41c6-86b5-8e8255fd51dd)


### Adım 4: Yeni Verilerle Tahmin (Inference)

Son olarak, eğitilen modelin pratik hayatta nasıl çalışacağını test etmek için daha önce hiç görmediği yeni ürün başlıkları üzerinde tahminler yapıldı. Model, bu yeni veriler için de oldukça başarılı ve mantıklı kategoriler atadı.



---

## Teknik Detaylar ve Kurulum

### Gerekli Kütüphaneler
```bash
pip install pandas numpy scikit-learn nltk matplotlib seaborn jupyter
```

### Kurulum Adımları
1.  Proje dosyalarını bilgisayarınıza indirin.
2.  Terminal veya komut isteminde `pip` komutu ile yukarıdaki kütüphaneleri yükleyin.
3.  Bir Python ortamında aşağıdaki NLTK veri paketlerini indirin:
    ```python
    import nltk
    nltk.download('stopwords')
    nltk.download('wordnet')
    nltk.download('omw-1.4')
    ```
4.  Jupyter Notebook'u başlatın.
5.  Önce `turkwise-task-eda-ve-preprocess.ipynb` dosyasını çalıştırarak `balanced_shuffled_products.csv` dosyasını oluşturun.
6.  Ardından `tf-idf-log-reg-category-prediction.ipynb` dosyasını çalıştırarak modeli eğitin ve sonuçları inceleyin.

---

## Proje Dosyaları

*   `turkwise-task-eda-ve-preprocess.ipynb`: Veri setini birleştirme, keşifsel veri analizi, sınıf dengesizliğini analiz etme ve dengeli bir veri seti (`balanced_shuffled_products.csv`) oluşturma işlemlerini içerir.
*   `tf-idf-log-reg-category-prediction.ipynb`: Dengeli veri setini kullanarak metin ön işleme, TF-IDF vektörleştirme, Lojistik Regresyon modelini eğitme, değerlendirme ve yeni verilerle tahmin yapma adımlarını içerir.

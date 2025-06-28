# 1- Ürün Başlığından Otomatik Açıklama Üretimi

Bu proje, e-ticaret sitelerinde yer alan ürün başlıklarını kullanarak, bu ürünler için otomatik, yaratıcı ve pazarlama odaklı açıklamalar oluşturmayı amaçlamaktadır. Projede, Hugging Face platformu üzerinden erişilen `google/gemma-2b` modeli, "One-Shot Learning" tekniği ile kullanılmıştır.

## Projenin Amacı

E-ticaret platformlarında her bir ürün için manuel olarak çekici ve bilgilendirici açıklamalar yazmak zaman alıcı bir süreçtir. Bu projenin temel amacı, bu süreci otomatize ederek verimliliği artırmaktır. Bir ürün başlığı verildiğinde, sistemin o ürünün özelliklerini ve faydalarını vurgulayan, satışa yönelik bir metin üretmesi hedeflenmektedir.

## Kullanılan Teknolojiler ve Yöntemler

- **Programlama Dili:** Python
- **Kütüphaneler:**
  - `transformers`: Hugging Face modellerini ve tokenizer'larını yüklemek ve kullanmak için.
  - `torch`: Modelin çalışması için gereken backend kütüphanesi.
  - `pandas`: Veri setini okumak ve yönetmek için.
  - `accelerate`: Modellerin daha verimli yüklenmesini ve çalışmasını sağlamak için.
- **Model:** Google tarafından geliştirilen `google/gemma-2b` modeli kullanılmıştır. Bu model, metin üretme konusundaki yetenekleri ve görece daha küçük boyutuyla verimli bir seçenektir.
- **Yöntem: One-Shot Prompting**
  Modelin istenen formatta ve stilde bir çıktı üretmesi için "One-Shot Learning" adı verilen bir prompt mühendisliği tekniği kullanılmıştır. Bu teknikte, modele asıl görev verilmeden önce, doğru bir örnek (`Örnek Başlık -> Örnek Açıklama`) sunulur. Bu sayede model, çıktının nasıl bir yapıda olması gerektiğini öğrenir.

  Kullanılan prompt şablonu şu şekildedir:
  ```
  **Role:** You are an expert e-commerce copywriter. Your goal is to write a captivating and persuasive product description based on the given product title.

  **Task:** Write a product description for the given product title, following the style and format of the examples.

  **Example:**
  **Product Title:** "AeroFlow Pro 5-in-1 Ionic Hair Dryer with Diffuser Attachment"
  **Product Description:** "Achieve a flawless, salon-quality blowout at home with the AeroFlow Pro, which uses advanced ionic technology..."

  **Product Title:** "{YENİ_ÜRÜN_BAŞLIĞI}"
  **Product Description:**
  ```

## Kurulum ve Çalıştırma

1.  **Gerekli Kütüphanelerin Yüklenmesi:**
    ```bash
    pip install --upgrade transformers accelerate pandas torch
    ```
2.  **Hugging Face Girişi:**
    Modeli indirebilmek için kod içerisinde `notebook_login()` fonksiyonu ile Hugging Face hesabınıza giriş yapmanız gerekmektedir. Bu, Kaggle veya Colab ortamlarında "Secrets" bölümüne token'ınızı ekleyerek yapılır.
3.  **Kodun Çalıştırılması:**
    Python betiği veya Jupyter Notebook çalıştırıldığında, `products-6000/balanced_shuffled_products.csv` dosyasından ürün başlıklarını okur, `gemma-2b` modelini yükler ve her bir başlık için sırayla açıklama üretir.

## Önemli Çıktılar ve Değerlendirme

Model, çoğu zaman ürün başlığındaki anahtar kelimeleri başarıyla anlayıp ilgili ve tutarlı açıklamalar üretmiştir. Ancak bazı durumlarda "halüsinasyon" olarak bilinen, konuyla tamamen alakasız metinler ürettiği de gözlemlenmiştir.

### Başarılı Çıktı Örnekleri

Bu örneklerde model, ürünün ne olduğunu doğru bir şekilde anlamış ve e-ticaret formatına uygun, ikna edici metinler oluşturmuştur.

**Örnek 1 (Oyun Konsolu Güç Adaptörü):**
> **Orijinal Başlık:** Power Supply for PS2, Replacement AC Adapter Charger Cord for Sony Playstation 2 PS2 Slim A/C 70000 Gaming Console
>
> **Üretilen Açıklama:** "The Power Supply for PS2 is a replacement AC adapter charger cord for Sony Playstation 2 PS2 Slim A/C 70000 Gaming Console. This high-quality power supply features a built-in surge protector to protect your console from power surges and other electrical disturbances. It also has a built-in safety fuse to prevent overheating and short-circuiting. The Power Supply for PS2 is compatible with all models of the Sony Playstation 2 PS2 Slim A/C 70000 Gaming Console, and it comes with a 1-year warranty for added peace of mind. Order now and enjoy uninterrupted gaming with the Power Supply for PS2."

**Örnek 2 (Bebek Partisi Oyunu):**
> **Orijinal Başlık:** Diaper Raffle Game Sign and Card, Carnival Circus Baby Shower Party Game Set, 1 Sign with 50 Raffle Tickets – DR04
>
> **Üretilen Açıklama:** "This Diaper Raffle Game Sign and Card Set is the perfect addition to any baby shower party! The sign features a cute circus theme and includes 50 raffle tickets, perfect for a diaper raffle game. The sign is made of durable cardstock and measures 11.5" x 17.5", making it easy to display and use. The raffle tickets are also made of durable cardstock and feature a fun circus design. This set is sure to be a hit at any baby shower party, and will help make the diaper raffle game a fun and memorable part of the celebration."

### İyileştirme Gerektiren Çıktılar (Halüsinasyon)

Bu örneklerde model, ürün başlığından tamamen bağımsız, alakasız ürünler hakkında açıklamalar üretmiştir. Bu durum, modelin bazen girdideki bağlamı kaçırabildiğini göstermektedir.

**Örnek 1 (Oyun Kumandası -> Dijital Pazarlama Kitabı):**
> **Orijinal Başlık:** StrikerDC Dreamcast Controller - Gray
>
> **Üretilen Açıklama:** "Are you looking to take your digital marketing skills to the next level? Look no further than The Ultimate Guide to Digital Marketing! This comprehensive"

**Örnek 2 (Klavye Tuş Takımı -> Spor Ayakkabı):**
> **Orijinal Başlık:** HK GAMING Custom Keycaps | Dye Sublimation PBT Keycap Set for Mechanical Keyboard | 139 Keys | Cherry Profile | ANSI US-Layout | Compatible with Cherry MX, Gateron, Kailh, Outemu | Stealth Dolch
>
> **Üretilen Açıklama:** "K-Swiss Men's Classic Court Sneaker is a classic sneaker with a modern twist. The sneaker features a durable leather upper, a cushioned midsole for comfort, and a rubber outsole for traction. The sneaker is available in a variety of colors and sizes, making it a versatile and stylish addition to any wardrobe."

## Sonuç ve Gelecek Çalışmalar

Bu proje, `gemma-2b` gibi büyük dil modellerinin, doğru prompt mühendisliği teknikleriyle e-ticaret metinleri üretme konusunda ne kadar yetenekli olabileceğini göstermektedir. One-shot learning, hızlı ve etkili sonuçlar verse de, halüsinasyon sorununu tamamen ortadan kaldırmamaktadır.

**Olası İyileştirmeler:**
1.  **Fine-Tuning:** Modeli, (başlık, açıklama) çiftlerinden oluşan daha büyük bir veri setiyle eğitmek (fine-tuning), modelin tutarlılığını artırabilir ve halüsinasyonları önemli ölçüde azaltabilir.
2.  **Daha Büyük Modeller:** `gemma-7b` gibi daha büyük parametreli modeller kullanarak daha tutarlı ve yaratıcı sonuçlar elde edilebilir.

---

### **Üretken Model Çıktıları Analizi ve Karşılaştırma Raporu**

#### **Kategori 1: Yüksek Başarımlı ve Tutarlı Çıktılar**

Bu kategorideki örnekler, modelin kendisine verilen görevi başarıyla yerine getirdiğini göstermektedir. Model, ürün başlığındaki anahtar kelimeleri, markayı ve ürün özelliklerini doğru bir şekilde ayrıştırarak bunları akıcı ve ikna edici bir pazarlama metnine dönüştürmüştür.

*   **Örnek 2 (PS2 Güç Adaptörü) ve Örnek 8 (Nintendo Switch Bataryası):** Bu teknik ürünlerde model, model numaraları (`HAC-003`), kapasite (`4400mAh`) gibi spesifik detayları metne doğru bir şekilde entegre etmiştir. Ayrıca, başlıkta yer almayan ancak ürünle ilgili olan "dahili sigorta" veya "orijinal bataryadan daha uzun ömür" gibi değer katan faydaları da ekleyerek metni zenginleştirmiştir. Bu, modelin bağlamsal çıkarım yeteneğinin yüksek olduğunu göstermektedir.

*   **Örnek 5 (Bebek Partisi Oyunu) ve Örnek 9 (Bebek Partisi Kuşağı):** Bu örneklerde model, ürünün temasını (sirk, orman) ve içeriğini (50 bilet, kuşak) anlayarak hedef kitleye yönelik sıcak ve davetkar bir dil kullanmıştır. Çıktılar, ticari amaçla doğrudan kullanılabilecek kalitededir.

*   **Değerlendirme:** İncelenen 10 örneğin 7'si (%70) bu kategoriye girmektedir. Bu, "one-shot" tekniğinin, net ve anlaşılır başlıklarda yüksek bir başarı oranına sahip olduğunu kanıtlamaktadır.

#### **Kategori 2: Model Hataları ve Tutarsızlıklar (Halüsinasyon)**

Bu kategorideki çıktılar, modelin girdi ile olan bağını tamamen veya kısmen kaybettiği durumları içermektedir. Bu hatalar iki alt tipte gözlemlenmiştir:

*   **Tamamen Bağlamsız Halüsinasyon:** Modelin, girdi olarak verilen ürün başlığıyla hiçbir ilgisi olmayan içerikler üretmesidir.
    *   **Örnek 1 (Oyun Kumandası):** Girdi bir "Dreamcast oyun kumandası" iken, çıktı bir "dijital pazarlama rehberi" hakkındadır. İki konu arasında hiçbir mantıksal bağ bulunmamaktadır.
    *   **Örnek 4 (Klavye Tuş Takımı):** Çok sayıda teknik detay içeren klavye tuş takımı başlığına karşılık, model bir "spor ayakkabı" açıklaması üretmiştir. Bu, girdinin karmaşıklığının modeli yanlış yönlendirebileceğinin bir göstergesidir.

*   **Bağlamsal Hata:** Modelin, ürünün genel kategorisini doğru anlamasına rağmen detaylarda hatalı bilgi üretmesidir.
    *   **Örnek 7 (Zelda Ocarina of Time 3D):** Model, ürünün bir "Zelda" oyunu olduğunu doğru tespit etmiştir. Ancak açıklamanın içine, bu serinin farklı bir oyununa (*Breath of the Wild*) ait olan "Sheikah Slate" mekaniğini dahil etmiştir. Bu durum, modelin konularla ilgili genel bilgiye sahip olduğunu ancak spesifik gerçekleri karıştırabildiğini göstermektedir.

**Sonuç**

`google/gemma-2b` modeli, "one-shot" yönlendirme ile dahi e-ticaret metinleri üretme konusunda kayda değer bir potansiyel sergilemektedir. Sistemin başarı oranı %70 olarak ölçülmüş olup, başarılı çıktılar ticari kullanıma uygun niteliktedir.

Bununla birlikte, %30'luk bir oranda gözlemlenen ve tamamen alakasız içerik üretimiyle sonuçlanan **halüsinasyon sorunu**, sistemin mevcut haliyle denetimsiz bir şekilde kullanılmasının önündeki en büyük engeldir. Özellikle karmaşık veya çok fazla teknik terim içeren başlıklarda modelin hata yapma eğilimi artmaktadır.

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

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

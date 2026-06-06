---
external: false
draft: false
title: "macOS İçin GoodbyeDPI Alternatifi: Türkiye’de Discord ve Roblox Yasaklarını Tek Tıkla Aşın (VPN’siz)"
description: "Discord ve Roblox gibi platformlara getirilen yasakları aşmak için macOS'ta Zapret-TR kullanımı ve DNS zehirlenmesi çözümü."
date: 2026-06-06
---

Discord ve Roblox gibi platformlara getirilen yasakları aşmak için Windows tarafında popüler olan **GoodbyeDPI** aracını duymuşsunuzdur. Ancak macOS kullanıcıları için bu aracın doğrudan bir alternatifi bulunmuyor. VPN kullanmak ise çoğu zaman internet hızını düşürüyor ve ping sorunları yaratıyor.

Bu yazıda, bilinen açık kaynaklı DPI aşma aracı olan **Zapret** projesini baz alarak macOS için özel olarak yapılandırdığımız [**Zapret-TR**](https://github.com/Hajorda/zapret-tr) (Zapret fork'u) kurulumundan bahsedeceğiz. Zapret-TR, GoodbyeDPI ile aynı mantıkta çalışarak internet hızınız düşmeden yasaklı sitelere tek komutla erişmenizi sağlar.

---

## DPI (Derin Paket İnceleme) ve DNS Zehirlenmesi Nedir?

Çözüme geçmeden önce, sorunun kaynağını anlamak çok önemli. İnternet servis sağlayıcıları (Superonline, Türk Telekom vb.) engelli sitelere girmenizi iki yöntemle durdurur:

1. **DNS Zehirlenmesi (DNS Hijacking):** Siz `discord.com`'a girmek istediğinizde, servis sağlayıcınız araya girip sizi sahte bir "Bu site BTK kararıyla engellenmiştir" sayfasına yönlendirir. Bu durumda tarayıcınız "Bağlantınız Gizli Değil" (`ERR_CERT_AUTHORITY_INVALID`) hatası verir.
2. **DPI (Derin Paket İnceleme):** Bağlantınızın başlığını gizlice okuyup yasaklı kelimeyi (`discord.com` veya `roblox.com`) gördükleri anda bağlantınızı anında koparırlar. ("Bağlantı Sıfırlandı" hatası alırsınız).

İşte biz bu rehberde, bu iki engeli de tamamen ortadan kaldıracağız.

---

## 1. Adım: DNS Zehirlenmesini Aşmak (Zorunlu Adım)

Zapret-TR'yi kurmadan önce mutlaka DNS engelini aşmamız gerekiyor. Bunun için tarayıcınızın veya sisteminizin **Güvenli DNS (DNS-over-HTTPS)** özelliğini aktif etmelisiniz.

![Güvenli DNS Ayarı](/images/zapret/zapret0.png)

**Nasıl Yapılır?**
* **Chrome / Brave / Arc:** Ayarlar > Gizlilik ve Güvenlik > Güvenlik > "Güvenli DNS Kullan" seçeneğini açıp "Cloudflare (1.1.1.1)" seçin.
* Bu işlemi yaptığınızda artık DNS üzerinden kandırılmayacaksınız ve sitenin "gerçek" adresini bulabileceksiniz.

---

## 2. Adım: macOS İçin Zapret-TR Kurulumu (Tek Tıkla!)

Sıra geldi o kırılamaz denen DPI (Derin Paket İnceleme) duvarlarını parçalamaya. Bunun için GitHub üzerinde açık kaynak kodlu olarak geliştirdiğimiz **Zapret-TR** kurulum scriptini kullanacağız.

Mac'inizdeki arama (Spotlight - `Cmd + Space`) çubuğuna **Terminal** yazıp uygulamayı açın ve aşağıdaki tek satırlık komutu yapıştırıp `Enter`'a basın:

```bash
curl -fsSL https://raw.githubusercontent.com/Hajorda/zapret-tr/master/install.sh | sudo bash
```

Sizden Mac şifrenizi (bilgisayarı açarken girdiğiniz şifre) isteyecektir. Şifrenizi yazarken ekranda harfler görünmez, bu normaldir; yazıp `Enter`'a basın.

![Kurulum Tamamlandı](/images/zapret/zapret1.svg)

---

## 3. Adım: Yasakları Aşmaya Başlamak

Kurulum bittikten sonra script size kullanabileceğiniz 4 farklı seçenek sunan interaktif bir menü açacaktır.

![Zapret-TR Menüsü](/images/zapret/zapret2.svg)

**Seçenekler Ne İşe Yarıyor?**
* **1, 2 veya 3 (Geçici Çalıştırma):** Eğer sadece ara sıra Discord'a veya Roblox'a girecekseniz, menüden **1**'i seçin. Terminal açık kaldığı sürece engeller aşılacaktır. İşiniz bitince Terminali kapattığınızda sistem normal haline döner. *(Not: Eğer 1. seçenek çalışmazsa ISS'niz farklı bir sistem kullanıyor demektir, 2. veya 3. seçeneği deneyin).*
* **4 (Arkaplan Servisi):** "Ben her seferinde kod yazmak istemiyorum" diyorsanız 4'ü seçin. Program, Mac'iniz her açıldığında arkada gizli bir "hayalet" gibi otomatik çalışacak şekilde kendini kurar. Kur bir daha unut!

Artık Discord'un sesli sohbetlerine girebilir, Roblox'u VPN'siz ve düşük ping ile oynayabilir, Pastebin gibi sitelere geliştirici olarak anında erişebilirsiniz.

---

## Nasıl Kaldırılır?

Eğer programı bilgisayarınızdan tamamen kazımak ve ağ ayarlarınızı orijinal, el değmemiş ilk günkü haline sıfırlamak isterseniz, tek yapmanız gereken Terminal'e şu komutu girmektir:

```bash
curl -fsSL https://raw.githubusercontent.com/Hajorda/zapret-tr/master/uninstall.sh | sudo bash
```

![Kaldırma İşlemi Başlıyor](/images/zapret/zapret3.svg)

## Sonuç

VPN kullanmak tüm bilgisayarınızın internet trafiğini yurtdışına gönderdiği için pinginizi uçurur ve bağlantınızı felç eder. Açık kaynak kodlu ve tamamen ücretsiz olan **Zapret-TR** gibi araçlar sayesinde ise, yalnızca yasaklı sitelere giden veri paketleriniz manipüle edilir. Böylece internet hızınızda %1'lik bir düşüş bile yaşamazsınız.

Projenin açık kaynak kodlarına, GitHub deposuna ve detaylarına buradan ulaşabilir, yıldıza basarak projeye destek olabilirsiniz:
👉 [https://github.com/Hajorda/zapret-tr](https://github.com/Hajorda/zapret-tr)

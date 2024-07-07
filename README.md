* Biliyorsunuz Airchain son zamanlarda sürekli hata veriyor. Az önce bir şeyler denedim ve oto-restart sorunsuz çalşıyor. (Kontrol edildi)
* Çayınızı kahvenizi aldıysanız, hadi başlayalım.

* İlk olara Cron tablosunu açalım.

```console
crontab -e
```

* En alt kısma bu komutu ekleyelim. 
# (*/10 kısmı bizim dakikamız. Ben 10 dakika olarak belirledim. İstediğiniz dakikayı belirleyebilirsiniz, özgürsünüz. (Muhtemelen😁) Ctrl+x - Ctrl+y - Enter

```console
*/10 * * * * sudo systemctl stop tracksd && sudo systemctl daemon-reload && sudo systemctl enable tracksd && sudo systemctl restart tracksd && sudo journalctl -u tracksd -fo cat
```

<img width="1304" alt="Ekran Resmi 2024-07-07 10 12 33" src="https://github.com/kaplanbitcoin1/AirchainRollup-OtoRestart/assets/98455323/575d05f8-fb98-4ef8-ad76-49980f629e09">


* Şimdi Sudoers Dosyasındaki Değişiklikleri oluşturmamız gerekiyor. Bunun için öncelikle Hostname öğrenelim. (Bu zaten sunucunuzun ismi. Sunucu oluştururken isim vermediyseniz size rastgele bir isim atanmıştır, bunu öğrenelim. (Aslında sunucuya giriş yaptığımızda görünüyor)



```rust
hostname
```

<img width="649" alt="Ekran Resmi 2024-07-07 10 05 50" src="https://github.com/kaplanbitcoin1/AirchainRollup-OtoRestart/assets/98455323/217ad917-5798-448b-8588-f13ea17bdfa8">

* Benim sunucu ismim 'vmi1949362'
* Sudoers dosyasını düzenlemek için:

```console
sudo visudo
```

* En aşağı kısma gelelim ve kodu yapıştıralım. (Sunucuismin kısmını kendi bilgilerinizle değiştirelim. (Parantezler 🧨 😁) Ctrl+x - Ctrly+y - Enter

```console
(Sunucuismin) ALL=(ALL) NOPASSWD: /bin/systemctl stop tracksd, /bin/systemctl daemon-reload, /bin/systemctl enable tracksd, /bin/systemctl restart tracksd, /bin/journalctl -u tracksd -fo cat
```




* Cron Servisinin Çalışıp Çalışmadığını Kontrol Etme. (Görseldeki gibi Active olmalı.)

```console
sudo systemctl status cron
```

<img width="784" alt="Ekran Resmi 2024-07-07 10 23 54" src="https://github.com/kaplanbitcoin1/AirchainRollup-OtoRestart/assets/98455323/5d236760-9dd8-4fc7-84cf-aa4123f17d86">


* Eğer servis Active değilse, başlatmak için:


```console
sudo systemctl start cron
```


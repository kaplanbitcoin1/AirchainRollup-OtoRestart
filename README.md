> Biliyorsunuz Airchain son zamanlarda sürekli hata veriyor. Az önce bir şeyler denedim ve oto-restart sorunsuz çalşıyor. (Kontrol edildi) Çayınızı kahvenizi aldıysanız, hadi başlayalım.

* İlk olara Cron tablosunu açalım.

```console
crontab -e
```

* En alt kısma bu komutu ekleyelim. 
> (*/10 kısmı bizim dakikamız. Ben 10 dakika olarak belirledim. İstediğiniz dakikayı belirleyebilirsiniz, özgürsünüz. (Muhtemelen😁) Ctrlx-Ctrly-Enter

```rust
*/10 * * * * sudo systemctl stop tracksd && sudo systemctl daemon-reload && sudo systemctl enable tracksd && sudo systemctl restart tracksd && sudo journalctl -u tracksd -fo cat
```

<img width="1536" alt="Ekran Resmi 2024-07-07 09 55 59" src="https://github.com/kaplanbitcoin1/AirchainRollup-OtoRestart/assets/98455323/92d7fd5e-fdd2-440a-b174-bf3fd0900eac">


* Şimdi Sudoers Dosyasındaki Değişiklikleri oluşturmamız gerekiyor. Bunun için öncelikle Hostname öğrenelim. (Bu zaten sunucunuzun ismi. Sunucu oluştururken isim vermediyseniz size rastgele bir isim atanmıştır, bunu öğrenelim. (Aslında sunucuya giriş yaptığımızda görünüyor)



```rust
hostname
```

<img width="649" alt="Ekran Resmi 2024-07-07 10 05 50" src="https://github.com/kaplanbitcoin1/AirchainRollup-OtoRestart/assets/98455323/217ad917-5798-448b-8588-f13ea17bdfa8">

* Benim sunucu ismim 'vmi1949362' Sadece bu kısmı ekleyeceğiz Sudoers dosyasına. Bunun için diyelim.

```rust
sudo visudo
```

* En aşağı kısma gelelim ve kodu yapıştıralım. (Sunucuismin kısmını kendi bilgilerinizle değiştirelim. (Elbette Parantezleri Uçuralım)

```rust
(Sunucuİsmin) ALL=(ALL) NOPASSWD: /bin/systemctl stop tracksd, /bin/systemctl daemon-reload, /bin/systemctl enable tracksd, /bin/systemctl restart tracksd, /bin/journalctl -u tracksd -fo cat
```

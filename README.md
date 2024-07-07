> Biliyorsunuz Airchain son zamanlarda sürekli hata veriyor. Az önce bir şeyler denedim ve oto-restart sorunsuz çalşıyor. (Kontrol edildi) Çayınızı kahvenizi aldıysanız, hadi başlayalım.

# İlk olara Cron tablosunu açalım.

```rust
crontab -e
```

# En alt kısma bu komutu ekleyelim. 
> (*/10 kısmı bizim dakikamız. Ben 10 dakika olarak belirledim. İstediğiniz dakikayı belirleyebilirsiniz, özgürsünüz. (Muhtemelen😁) Ctrlx-Ctrly-Enter

```rust
*/10 * * * * sudo systemctl stop tracksd && sudo systemctl daemon-reload && sudo systemctl enable tracksd && sudo systemctl restart tracksd && sudo journalctl -u tracksd -fo cat
```

<img width="1536" alt="Ekran Resmi 2024-07-07 09 55 59" src="https://github.com/kaplanbitcoin1/AirchainRollup-OtoRestart/assets/98455323/92d7fd5e-fdd2-440a-b174-bf3fd0900eac">


# Şimdi Sudoers Dosyasındaki Değişiklikleri oluşturmamız gerekiyor. Bunun için öncelikle Hostname öğrenelim. (Bu zaten sunucunuzun ismi. Sunucu oluştururken isim vermediyseniz size rastgele bir isim atanmıştır, bunu öğrenelim. (Zaten sunucuya giriş yaptığımızda görünüyor)



```rust
hostname
```

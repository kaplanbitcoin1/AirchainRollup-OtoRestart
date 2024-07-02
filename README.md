

> Öncelikle Core Node fedailerine teşekkürler 😁 İlk içeriği @eskola1 ve @brkcinar hazırlamış.

* 10 Dakika olan hata tespit durumunu 5 dakikaya indirdim çünkü Rollup genellikle 3-4 pod bastıktan sonra hataya düşüyor ve tekrar pod basmaya çalışıyor. Bu arada geçen süredeki denemeler sonuçsuz kalıyor ve 10 dakika yerine 5 dakikayı seçmemin nedeni bu: Fazladan beklememek. Duruma göre 2-3 dakikayı da deneyebiliriz 😳


* İlk olarak bu dosyanın içerisine girelim


```powershell
nano /root/check_tracks.sh
```

* Tek komut halinde içerisine yapıştıralım ve (Ctrlx+Ctrly=Enter) (Hiçbir şeyi değiştirmeden) 🐅

```cmd
# Log dosyasındaki hataları kontrol eden komutlar
vrf_init_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'Failed to Init VRF' | wc -l"
vrf_record_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'VRF record is nil' | wc -l"
vrf_validate_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'Failed to Validate VRF' | wc -l"
submitpod_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'ERR Error in SubmitPod' | wc -l"
verifypod_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'ERR Error in VerifyPod' | wc -l"
verifyhash_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'Failed to get transaction by hash: not found' | wc -l"

# Log dosyası kontrolü
check_log_file="/root/check_tracks.log"
check_log_command="find $check_log_file -mmin -5 | wc -l"
log_entry_count=$(eval $check_log_command)

echo "Son 5 dakika içinde log dosyasına eklenen girdi sayısı: $log_entry_count"

# Son 5 dakika içinde log dosyasına girdi eklenmediyse, servis durduruluyor ve yeniden başlatılıyor
if [ $log_entry_count -eq 0 ]; then
    echo "Son 5 dakika içinde log dosyasına girdi eklenmedi, servis durduruluyor ve yeniden başlatılıyor..."
    sudo systemctl stop tracksd
    sudo systemctl restart tracksd
    exit 0
fi

# Hata kontrolü ve işlemler
vrf_init_log_count=$(eval $vrf_init_log_command)
vrf_record_log_count=$(eval $vrf_record_log_command)
vrf_validate_log_count=$(eval $vrf_validate_log_command)
submitpod_log_count=$(eval $submitpod_log_command)
verifypod_log_count=$(eval $verifypod_log_command)
verifyhash_log_count=$(eval $verifyhash_log_command)

echo "Failed to Init VRF hatasının sayısı: $vrf_init_log_count"
echo "VRF record is nil hatasının sayısı: $vrf_record_log_count"
echo "Failed to Validate VRF hatasının sayısı: $vrf_validate_log_count"
echo "ERR Error in SubmitPod hatasının sayısı: $submitpod_log_count"
echo "ERR Error in VerifyPod hatasının sayısı: $verifypod_log_count"
echo "Failed to get transaction by hash: not found hatasının sayısı: $verifyhash_log_count"

if [ $vrf_init_log_count -ge 1 ] || [ $vrf_record_log_count -ge 1 ] || [ $vrf_validate_log_count -ge 1 ]; then
    echo "VRF hataları tespit edildi, servis durdurulacak ve yeniden başlatılacak..."

    sudo systemctl stop tracksd
    sudo systemctl restart tracksd

elif [ $submitpod_log_count -ge 3 ] || [ $verifypod_log_count -ge 3 ] || [ $verifyhash_log_count -ge 1 ]; then
    echo "SubmitPod ve VerifyPod hataları tespit edildi (3 veya daha fazla), servis durdurulacak, rollback yapılacak ve yeniden başlatılacak..."

    sudo systemctl stop tracksd
    /root/./tracks rollback
    /root/./tracks rollback
    /root/./tracks rollback
    sudo systemctl restart tracksd

else
    echo "Hata tespit edilmedi, servis pasif durumda."
fi

```

* Çalışma izni verelim

```console
chmod +x check_tracks.sh
```

* Dosya içine girelim

```console
crontab -e
```


* Bu komutu içerisine ekleyelim. (Eğer görseldeki gibi komutlar varsa (ki olması muhtemel) en alt kısma # ekleyip komutu yanına yapıştıralım. Eğer hiçbir şey olmadıysa bile bir şeyler olmuş olabilir: Tek komut yapıştırıp (Ctrl+x + Ctrl+y= Enter)


```console
*/5 * * * * bash /root/check_tracks.sh >> /root/check_tracks.log 2>&1
```

<img width="488" alt="Ekran Resmi 2024-07-02 16 23 42" src="https://github.com/kaplanbitcoin1/AirchainRollup-OtoRestart/assets/98455323/2008b59b-c972-4861-9541-d56acca17696">


> Son olarak tek komut

```console
sudo systemctl stop tracksd 

systemctl restart tracksd

sudo journalctl -u tracksd -fo cat
```


> Sanırım? 😁 🐅

<img width="1536" alt="Done" src="https://github.com/kaplanbitcoin1/AirchainRollup-OtoRestart/assets/98455323/e2b1afa9-d799-4e40-8ef0-9b2b1566d859">



* Son Rötuşlar: Eğer hata aldıktan sonra Script'in 5 dakika'da restart yapması yerine, 5 defa hata aldığında restart yapmasını isterseniz, 'nano /root/check_tracks.sh' içerisine bu komutu yapıştırın ve kahvenizden bir yudum alın)  

```powershell
# Log dosyasındaki hataları kontrol eden komutlar
vrf_init_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'Failed to Init VRF'"
vrf_record_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'VRF record is nil'"
vrf_validate_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'Failed to Validate VRF'"
submitpod_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'ERR Error in SubmitPod'"
verifypod_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'ERR Error in VerifyPod'"
verifyhash_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'Failed to get transaction by hash: not found'"

# Log dosyası kontrolü
check_log_file="/root/check_tracks.log"
check_log_command="find $check_log_file -mmin -5 | wc -l"
log_entry_count=$(eval $check_log_command)

echo "Son 5 dakika içinde log dosyasına eklenen girdi sayısı: $log_entry_count"

# Son 5 dakika içinde log dosyasına girdi eklenmediyse, servis durduruluyor ve yeniden başlatılıyor
if [ $log_entry_count -eq 0 ]; then
    echo "Son 5 dakika içinde log dosyasına girdi eklenmedi, servis durduruluyor ve yeniden başlatılıyor..."
    sudo systemctl stop tracksd
    sudo systemctl restart tracksd
    exit 0
fi

# Hata kontrolü ve işlemler
vrf_init_errors=$(eval $vrf_init_log_command)
vrf_record_errors=$(eval $vrf_record_log_command)
vrf_validate_errors=$(eval $vrf_validate_log_command)
submitpod_errors=$(eval $submitpod_log_command)
verifypod_errors=$(eval $verifypod_log_command)
verifyhash_errors=$(eval $verifyhash_log_command)

echo "Son 5 hata (Failed to Init VRF):"
echo "$vrf_init_errors"
echo ""
echo "Son 5 hata (VRF record is nil):"
echo "$vrf_record_errors"
echo ""
echo "Son 5 hata (Failed to Validate VRF):"
echo "$vrf_validate_errors"
echo ""
echo "Son 5 hata (ERR Error in SubmitPod):"
echo "$submitpod_errors"
echo ""
echo "Son 5 hata (ERR Error in VerifyPod):"
echo "$verifypod_errors"
echo ""
echo "Son 5 hata (Failed to get transaction by hash: not found):"
echo "$verifyhash_errors"
echo ""

# Toplam hata sayısı kontrolü
total_errors=$((vrf_init_errors + vrf_record_errors + vrf_validate_errors + submitpod_errors + verifypod_errors + verifyhash_errors))

if [ $total_errors -ge 5 ]; then
    echo "Toplam 5 veya daha fazla hata tespit edildi, servis durdurulacak ve yeniden başlatılacak..."

    sudo systemctl stop tracksd

    # Pod hataları için rollback
    if [ $submitpod_errors -ge 3 ] || [ $verifypod_errors -ge 3 ] || [ $verifyhash_errors -ge 1 ]; then
        /root/./tracks rollback
        /root/./tracks rollback
        /root/./tracks rollback
    fi

    sudo systemctl restart tracksd

else
    echo "Hata tespit edilmedi veya hata sayısı yeterli değil, servis pasif durumda."
fi
```



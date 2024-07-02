* İlk olarak bu dosyanın içerisine girelim


```shell
nano /root/check_tracks.sh
```
* İçerisine tek komut olarak yapıştıralım

```shell
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

# Son 5 dakika içinde log dosyasına girdi eklenmediyse, servis yeniden başlatılıyor
if [ $log_entry_count -eq 0 ]; then
    echo "Son 5 dakika içinde log dosyasına girdi eklenmedi, servis yeniden başlatılıyor..."
    restart_command="sudo systemctl restart tracksd && sudo journalctl -u tracksd -fo cat"
    eval $restart_command
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

    stop_command="sudo systemctl stop tracksd"
    restart_command="sudo systemctl restart tracksd"

    eval $stop_command
    eval $restart_command

elif [ $submitpod_log_count -ge 3 ] || [ $verifypod_log_count -ge 3 ] || [ $verifyhash_log_count -ge 1 ]; then
    echo "SubmitPod ve VerifyPod hataları tespit edildi (3 veya daha fazla), servis durdurulacak, rollback yapılacak ve yeniden başlatılacak..."

    stop_command="sudo systemctl stop tracksd"
    rollback_command="/root/./tracks rollback && /root/./tracks rollback && /root/./tracks rollback"
    restart_command="sudo systemctl restart tracksd"

    eval $stop_command
    eval $rollback_command
    eval $restart_command

else
    echo "Hata tespit edilmedi, servis pasif durumda."
fi
```



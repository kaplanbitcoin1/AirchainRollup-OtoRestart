

> Öncelikle Core Node fedailerine teşekkürler 😁 İlk içeriği @eskola1 ve @brkcinar hazırlamış.

* 10 Dakika olan hata tespit durumunu 5 dakikaya indirdim çünkü Rollup genellikle 3-4 pod bastıktan sonra hataya düşüyor ve tekrar pod basmaya çalışıyor. Bu arada geçen süredeki denemeler sonuçsuz kalıyor ve 10 dakika yerine 5 dakikayı seçmemin nedeni bu: Fazladan beklememek. Duruma göre 2-3 dakikayı da deneyebiliriz 😳


* İlk olarak bu dosyanın içerisine girelim


```console
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




* Son Rötuşlar: Eğer hata aldıktan sonra Script'in 5 dakika'da restart yapması yerine, 5 defa hata aldığında restart yapmasını isterseniz, 'nano /root/check_tracks.sh' içerisine bu komutu yapıştırın ve kahvenizden bir yudum alın)  

```powershell
# Log dosyasındaki hataları kontrol eden komutlar
vrf_init_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'Failed to Init VRF' | wc -l"
vrf_record_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'VRF record is nil' | wc -l"
vrf_validate_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'Failed to Validate VRF' | wc -l"
submitpod_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'ERR Error in SubmitPod' | wc -l"
verifypod_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'ERR Error in VerifyPod' | wc -l"
verifyhash_log_command="journalctl -u tracksd -n 5 --no-pager | grep 'Failed to get transaction by hash: not found' | wc -l"
switchyard_error_command="journalctl -u tracksd -n 5 --no-pager | grep 'Switchyard client connection error' | wc -l"

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
switchyard_errors=$(eval $switchyard_error_command)

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
echo "Son 5 hata (Switchyard client connection error):"
echo "$switchyard_errors"
echo ""

# Toplam hata sayısı kontrolü
total_errors=$((vrf_init_errors + vrf_record_errors + vrf_validate_errors + submitpod_errors + verifypod_errors + verifyhash_errors + switchyard_errors))

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




```console
seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:25756"
```


```powershell
persistent_peers = "3f485e77240d0c47252fadbc95470315d10cd7e1@135.181.246.31:25756,f398117804c638ace3c189781d08db77caa20aa9@65.108.109.48:10656,c732a1ddbaccda161b60059a572e0fcba7828bc5@176.9.90.222:60556,c09a9d33c1edbfb9406612d45237a8a59e5c67ad@148.251.185.41:56116,0b108b734b37f3cee0217c6a65e8d5bff35fb45a@142.132.146.185:50156,64928cf0b8414a42e7f6cfd7f11cfaa7eb32d37d@78.46.76.134:25756,f69303010bb37c038841abc0b0259fa959e3e13d@49.12.174.132:15656,67138107cf95087b10efd99ae2a7d7e374e9ee84@18.200.93.175:26656,2c729d33d22d8cdae6658bed97b3097241ca586c@195.14.6.129:26019,7f9559787af4c1705bc1524c37a5dbc48994a905@166.1.173.230:42156,95071a39cab00a428c532c682c6cd05618f5a9f4@46.4.80.48:25756,e838948adde1779e16f70ebd7f1b46b38710bb22@207.188.6.109:26691,b41b9b0b398059bd483123043fafd1e311f05cbe@65.21.197.228:25756,3b944bcae9db0b88d8419adde8e26188a6a5ef5d@65.109.59.22:25756,07632ab562028c3394ee8e78823069bfc8de7b4c@37.27.52.25:19656,e44e11c6f229a571f4239781f249a25e4257c179@185.84.224.160:25756,f529f8321ac0422f02bc86b5a1d79e70f0f89616@64.185.227.242:25756,41453d8197207ce8834a672896eddf941f5b745e@125.131.208.75:26656,0940d8b2e54d7e435c26c4913a3c22862cb6ab9a@129.153.79.186:17956,10affdb68fb31c0fcbc4129bc113f73ef4a24636@148.113.153.132:26656,b2262b053c92e95c399076a992f4aaeabfe5d14c@89.116.28.50:11656,9c3cc16fbb1ff73ee98d9b5f0a098ac06fe58347@5.104.82.214:26656,33ca7e36ef9bbdb8abe8dd9fb7a28b104be04efc@167.86.98.202:26656,6a6d164766341e4e4f56d0359f130a757f21851a@95.217.148.179:29656,a10c8ae9fa2c564d574f5de42f6f814334720f66@84.247.170.82:15656,17b0fb616bae3fc2e6babf717e2ec132353142db@51.195.88.136:15674,5ed0f3e5c0cf6e287dfb53140ea5c379ae674945@168.119.91.28:44656,c5d1565f8ebbe3377f9195f03966da54ce3faaf5@194.163.158.180:26656,c36f8ae42381403d93e9be3fb637eb6c19c1ebce@46.4.87.147:4001,c99f1967ebcd5edb8eb9e624712e7372384a5917@57.129.28.107:46656,92ae44dadb21c468d17911ae82c4baab5878896f@31.220.80.169:26656,6c8c798a3583bbb42b52945b8f113495d94086a2@161.97.162.9:26656,66767c34b3250523b4bda94afde4e2c9dda081a1@84.247.178.68:26656,ab137f5c7eed1bb5172bd7cbe642ec17180ec397@193.34.213.155:33756,baea427d8a7d1e74d9897b3056b2dcb17b2edeca@65.108.98.235:53156,ad3070747236e72df81b6386bb9da9753fec240e@37.27.69.248:26656,bd943906483fafa2b783879f810ff8ed0b92d34a@88.198.17.120:33756,38ac96bc3299dc63301b113314aee2a4d356ffc3@88.198.46.55:25756,fc37e22ae9405cf00a775a014366d428376e47b3@37.27.48.77:29656,a63a6f6eae66b5dce57f5c568cdb0a79923a4e18@168.119.10.134:26628,0a62bff20aea40a69557c094e06bc4a7a68466d7@86.48.7.47:26656,16eec9be493fe9624cfb414bfc2cb06353f8d120@95.217.231.107:57300,ff9dbc6bb53227ef94dc75ab1ddcaeb2404e1b0b@162.55.65.140:26656,2da714585d9207ace5897d7a8506b78074cc5af5@65.108.70.106:26606,104e45f5843f6f2b5d16f9a046a4b96e7d79a235@65.109.83.40:25756,0e8a332206dc2c42dc8b48ed7ab26b4f76007535@144.76.14.158:20765,3abea9cfc98e7502877ea5a4b7a5068b7128b9d4@38.242.143.1:26656,a4b6849e58ff2b96ca35487a675db90e86134c51@88.99.137.151:26656,299616301b93c4f6c80e97d3d83d2ea66863c3ae@65.109.122.96:27656,c86acc6936b8cb85d75c68880092f99da8d7f380@5.9.85.89:22656,36c47cc7a1758163c577af0c2d2c5a23f8f79c47@15.235.160.23:26656,64424ee1f549a516afaf6669d1c4c775072837f2@74.118.139.219:26656,edcc2c7098c42ee348e50ac2242ff897f51405e9@65.109.34.205:36656,d4bd8277becfa688675829dc146ababa7a6d899f@65.109.89.5:36656,ac6e9e4162d1852c6150f1a67e405762d71afcb4@95.217.119.251:26656,d8e86fab476b44e8597064b6ab8cfb6c0701cfb1@149.50.110.154:19656,4e5f846406521563359ddae98fc0976ca8e1427a@38.242.135.52:27656,6cf83954175cd5569f8df38a5e5bd25bcb4b0cb4@86.48.7.48:26656,10b173a6c692fc1d258c10689e7adbc6d0b22ce3@162.55.242.213:55656,3897daecbbc96ef27c1a5c24db61cb4553d2aad7@86.48.7.41:26656,19552ae0bb71133327915cb9e2c46f3885d55284@148.251.179.244:26656,6a0ac37ec7b235e457a483dc57b2e73b8a77fd5e@51.210.222.92:26656,ef9a4ffa91280459a74070d9a08b8b03bc0dbc0e@88.99.3.143:26602,fc3329f887d5e11465976ae966cc7145eee8eff6@176.9.48.151:20756,d00b44e18f797672e4d18a81908559f4144e29da@220.85.94.217:26656,a76de25cb411a3be28911ca5b2879dd8405f47ab@54.210.75.170:26656,46531763cbc22ca800dc0e3022c650319184f16d@51.222.244.157:26656,4512eea4775c650616f08f30ba03ce68be2eac01@34.230.84.121:26656,3cefdca56dc5e72af817a54dfcfe5bb6fa65d1b3@92.118.56.200:26656,c04526a0a50bd2d40aa957b9d6d6b1a0492f133a@146.190.132.179:26656,7c56dcb1032b7eaae1b1276cd8066f792bd319fb@103.50.33.195:26656,e3e2a682aae8668c8bb0032256ba36d68b0b11b9@15.235.144.20:20656,0794060591ebcb457f00f1606f8360c0fa5740a4@86.48.3.114:26656,5cd075abbda4cc6d0c45bc7cc1cddb43ed0a5dca@23.88.65.98:26656,b8093934511a785d4d1a945156eec1a9f120e5d7@135.125.247.46:26656,7434a6d306784686af879dfdd620143ff2cea31a@193.34.213.225:14656,58b05aafa5a9685d4e5913b385ff4ff1894242a1@65.21.167.118:26656,3b0548ddd351920d03024ab8c58b36b443bd8737@65.109.19.126:15656,6dce33f4d2eabb163170169d5fa06e7bcae19c0d@50.242.73.9:62150,eab5dac731277edaf8634f3ea9cefafbb652dd72@69.67.150.147:25756,ab4b72f78bd3f9ba9e63735dcf9e269f59cedc67@136.243.44.52:26656,6fdb1418d5dc063bc36b4b3e8e4d91964649c982@162.55.25.233:26656,94a4bc7123d08aa98bf31dc34f458105b44de219@173.249.7.132:33756,f29cec230ce8287a1972af6d9463e2a224ad7c9d@89.117.61.95:14656,90a735bf21ee4364f3d021958d67915df52b431a@49.12.172.92:26656,bf53a6a7a417ef0d0e8d89845ae8a7d57eb87f63@142.132.128.137:26656,815788916095466ff3eebae5075b0e48a8da6aa4@89.58.38.59:25756,f9afd6428050908ea7fb9adad10e7f81c3e19e88@62.171.170.236:26656,c2e06165110ef3a7f834a2680a09a710a12124f5@75.119.157.58:14656,a1acfaccf76f6f45e9d1ff9dd595c27323fde494@185.239.209.222:26656,fcb78a22b17de6a407599224e112cfb7ec3dad21@148.113.9.181:26656,8ef2aa1ccf6035c700f47d0b7ce128b794e3d98f@161.97.147.188:26656,62775997caa3d814c5ad91492cb9d411aea91c58@51.38.53.103:26856,d2a6449106efe4fdd4918947431de767cc1f6dbd@194.163.158.58:26656,a98484ac9cb8235bd6a65cdf7648107e3d14dab4@95.217.74.22:17956,5c2a752c9b1952dbed075c56c600c3a79b58c395@46.4.25.224:26686,2b7dda5fa3e826c0b50d40cc3a6541e732ec6a76@65.109.103.43:13056,4f81f3565ecccfaa82fb52f5bc580eaea7594d4a@65.21.167.119:26656,3c5e76217271f37fb53c1500248c3148f47dacea@31.222.238.31:26656,369115f859e181ed513e7a71282fecff7b31483a@185.239.208.131:26656,d12c8e03a6137ae759b4589d7d2c85b56bc5fb62@38.242.230.55:25756,15dadcdb79ad05ef7b3cf8d0cc9cf270c005f191@57.128.74.22:56116,d8bc4c27661ec1c452ca38d01c0a3412d5248f79@84.247.137.128:26656,50c01d389c2e843f3218bed02b23d3af9eaae81a@223.67.16.4:27656,5aa89b1072b745ee816809df5b5797a04830ffe0@84.247.189.94:15656,2ca271324c313ebde4e179fb7579de468a2d3afe@135.181.232.227:14656,65e7473f79d3c19c4b4c5191a93d8d6365b5dc06@34.126.88.69:26656,3b1e9bb5426a702ed7f011d253504e89f4311987@158.220.98.71:26656,92a53312c43229ea0285fb9637d351e0a310c335@161.97.156.213:26656,af3e7211d71a070748dd2e1157cbdaa8f4a8e5a4@194.163.137.223:26656"
```


Для данного действия нам потребуется:  
раздел в файловой системе ext4  
образ дистрибутива  
  
На подготовленный раздел скачиваем образ дистрибутива Manjaro  
  
далее в файл /boot/grub/custom.cfg добавляем следующий текст
```sh
menuentry "Manjaro  grub_iso"  {    
set isofile="/manjaro-gnome-20.1-stable-x86_64.iso"    
set dri="free"    
set lang="en_US"    
set keytable="us"   
set timezone="Europe/Moscow"    
search --no-floppy -f --set=root $isofile  
probe -u $root --set=abc    
set pqr="/dev/disk/by-uuid/$abc"    
loopback loop $isofile  
linux  (loop)/boot/vmlinuz-x86_64  img_dev=$pqr img_loop=$isofile driver=$dri tz=$timezone lang=$lang keytable=$keytable copytoram  
initrd  (loop)/boot/intel_ucode.img (loop)/boot/initramfs-x86_64.img  
}
```

сохраняем, делаем sudo update-grub и перезагружаемся.  
  
при следующей загрузке в меню будет пункт Manjaro grub_iso  
возможно будут выскакивать ошибки, но в конечном итоге образ загрузится.
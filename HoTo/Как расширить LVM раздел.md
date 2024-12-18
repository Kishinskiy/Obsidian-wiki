
тебе надо sda3 предварительно расширить из этих ресурсов, через parted (https://gcore.com/learning/manage-disk-partitions-linux-parted-command/)

sudo parted /dev/sda
Fix
quit
sudo pvresize /dev/sda3
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv


Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.box_version = "1802.01"
  config.vm.provider "virtualbox" do |vb|
    file_to_disk = 'disk.vdi'
    unless File.exist?(file_to_disk)
      vb.customize ['createhd', '--filename', file_to_disk, '--size', 40 * 1024]
    end
    vb.customize ['storageattach', :id, '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
    vb.memory = "2048"
  end

  config.vm.provision "shell", inline: <<-SHELL
    yum install systat cryptsetup vim mdadm -y
    mkdir /tmp/boot
    cp -r /boot/* /tmp/boot/
    umount /boot
    sfdisk -d /dev/sda > backup.txt
    sfdisk /dev/sdb < backup.txt
    mdadm --create /dev/md0 --level=1 --raid-devices=2 --metadata=1.0 missing /dev/sdb2
    sleep 5
    mkfs.xfs /dev/md0 -f
    mount /dev/md0 /boot
    cp -r /tmp/boot/* /boot
    umount /boot
    mdadm --add /dev/md0 /dev/sda2
    sed -i '/\\/boot/d' /etc/fstab
    echo "UUID=$(blkid -s UUID /dev/md0 | awk -F'\"' '{print $2}') /boot xfs defaults 0 0" >> /etc/fstab
    mount -a
    pvcreate /dev/sdb3
    vgextend VolGroup00 /dev/sdb3
    swapoff -a
    lvreduce -l -2 /dev/VolGroup00/LogVol01 -f
    mkswap -f /dev/VolGroup00/LogVol01
    swapon -a
    lvconvert --type raid1 -m1 /dev/VolGroup00/LogVol00 -y
    lvconvert --type raid1 -m1 /dev/VolGroup00/LogVol01 -y
    echo "dm-raid" >> /etc/modules-load.d/dm-raid.conf
    mdadm --examine --scan >> /etc/mdadm.conf
    dracut -f -v --mdadmconf
    grub2-mkconfig -o /boot/grub2/grub.cfg
    grub2-install /dev/sdb
  SHELL
end
#lvm vgchange -ay --config ' global {locking_type=1} '

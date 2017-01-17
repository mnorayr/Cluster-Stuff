# Cluster-Stuff

-- Secure boot enable legacy support
-- Boot into Windows
-- Set everything off and create password for Windows
-- Shrink partition by 655092 MB
-- Reboot into Clonezilla and enter shell
	sudo fdisk -l
	sudo fdisk /dev/sda
	p
	n
	(enter partition 6) (enter first sector) last sector +120G
	p
	n
	(enter partition 7) (enter first sector) (enter last sector)
	w
	sudo mkswap /dev/sda6
	sudo mkfs.ext3 /dev/sda7
	

--Format nvme drives and mount in fstab
	fdisk -l
	fdisk /dev/nvme0n1
	p
	d (delete)
	p
	d
	p
	g (creates new disk label (GPT))
	n
	(enter) (enter) (enter)
	p
	w (write)
	fdisk -l
	
	fdisk /dev/nvme1n1 -- do same thing
	
	sudo mkfs.ext3 /dev/nvme0n1p1
	sudo mkfs.ext3 /dev/nvme1n1p1
	exit
	
--Start Clonezilla
	device-device
	beginner
	part_to_local_part
	choose sources sdb3 490G_ext3
	choose target sda7
	(enter) (enter) (enter)
	y
	y
	(wait)
	enter command line prompt
	
--Edit fstab with correct UUIDs
	df -h
	sudo mkdir /mnt/sda7
	sudo mount /dev/sda
	sudo mount /dev/sda7 /mnt/sda7
	more /mnt/sda7/etc/fstab
	sudo mkdir /mnt/boot-sda
	sudo mkdir /mnt/boot-sdb
	sudo mount /dev/sda1 /mnt/boot-sda
	sudo mount /dev/sdb2 /mnt/boot-sdb
	sudo cp -R /mnt/boot-sdb/EFI/centos/ /mnt/boot-sda/EFI/
	ls -al /mnt/boot-sda/EFI/centos
	poweroff

--reboot and disable Legacy
--reboot and boot into CentOS
	mkswap /dev/sda6
	swapon /dev/sda6
	lsblk -no UUID /dev/sda6 >> /etc/fstab
	nano /etc/fstab
		cut and paste UUID into correct spot (C-k C-u)
	lsblk -no UUID /dev/sda1 >> /etc/fstab
	nano /etc/fstab
		cut and paste for /boot/efi

--reboot
	sudo e2label /dev/sda7 /
	sudo nano /etc/sysconfig/network
	
	e2label /dev/nvme0n1p1 /mnt/disk1/
	e2label /dev/nvme1n1p1 /mnt/disk2/
	e2label /dev/sda7 /

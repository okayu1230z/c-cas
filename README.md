# C-CAS

## Description

This is C-CAS (Compact-CameraAndStorage), which provides cameras, storage, and even a web service to check the data. Based on this manual, you can inexpensively build a surveillance camera system for which you want historical data. All you need is as simple as a single Linux machine (e.g. Ubuntu, Raspberry Pi OS), a camera (e.g. sensor type, USB connected), and storage (e.g. SSD, HDD).

## Equipment needed

- Linux machine: Raspberry Pi4, ant others
- OS: Ubuntu 18.04 LTS, 20.04 LTS, 21.04
- Camera: USB connected type, sensor type, [example item](https://www.amazon.co.jp/gp/product/B08QYZ91D2)
- Storage: SSD, HDD, [example item](https://www.amazon.co.jp/dp/B07JG8NN8R)

The test environment is Ubuntu 20.04 on Raspberry Pi4, SSD with SATA connection, camera with USB connection.

## Setup

1. Check your environment

```
$ sudo apt install v4l-utils
```

With the camera and SSD connected to the Ubuntu machine, check the device information.
Check camera information: \<Device_name\> (-> /dev/video0)

```
$ v4l2-ctl --list-devices
$ v4l2-ctl -d /dev/video1 --info
```

Check storage information: \<Disk_name\> (-> /dev/sda)
C-CAD allows you to define the auto-mount service, so you don't need to mount the USB-connected SSD.

```
$ sudo fdisk -l
```

Create a directory for storage: \<Mount_point\> (-> /mnt/ssd1)

```
$ sudo mkdir /mnt/ssd1
$ sudo mkfs -t ext4 /dev/sda
```

Check username: \<User_name\> (-> user)

```
$ whoami
user
```

2. Download

```
$ sudo apt install motion git
$ git clone <this repository>
```

3. Setup

Configure the necessary items in the `/etc/motion/motion.conf` file. Rewrite it according to your environment.

- videodevice /dev/video0
- target_dir /mnt/ssd1
- width 640
- height 480
- framerate 15
- threshold 1500
- movie_output on
- movie_filename movie/%Y%m%d/%H%M%S
- movie_codec mp4

If you want to save the image
- snapshot_filename picture/%Y%m%d/%H%M%S
- snapshot_interval 10

Access from other than localhost
- webcontrol_localhost off
- stream_localhost off

Describe device name(`/dev/sda`) and mount point(`/mnt/ssd1`) in `system/storage-mount.service`.

```
[Unit]
Description=SSD Mount Service
Before=network.target

[Service]
ExecStart=mount -t ext4 <Device_name> <Mount_point>

[Install]
WantedBy=multi-user.target
```

Describe User name(`user`) in `system/rm7days.service`.

```
[Unit]
Description=rm 7days ago videos

[Service]
Type=simple
ExecStart=

[Install]
WantedBy=multi-user.targe
```

After the configuration is written, copy it to your system directory.

```
$ sudo cp system/* /etc/systemd/system
$ systemctl daemon-reload
```

docker setup

```
$ sudo apt install docker docker-compose
$ sudo usermod -aG docker $USER
```

## Management

1. Start C-CAS service

```
$ sudo systemctl start storage-mount
$ sudo systemctl start motion
```

If you want to view C-CAS from the web,


2. Maintenance

```
$ cd c-cas/httpd; docker-compose up -d
```

View C-CAS from the web -> `http://<ip_address>:8082`

If you want to use auto remove data from ssd/hdd,
you can use system/rm7days.service, rm7days.timer

Motion Control is at `http://<ip_address>:8080`

Streaming is at `http://<ip_address>:8081`

## (Optional) USB Temp 

Equipment needed
- [USB TEMPer](https://www.amazon.co.jp/gp/product/B004FI1570)

Articles to help with installation
- https://creepfablic.site/2019/09/28/raspberrypi-temper-2/
- https://ginkyo.hatenablog.jp/entry/2018/05/20/222627

```
$ sudo tempered 2> /dev/null | awk 'NR==1' | cut -b 17-
```

You can also put the current user as root.

```
okmt ALL=NOPASSWD: /usr/local/bin/tempered
```

## Contribution

It would be helpful if you could let us know via Issue or email if you encounter any problems as we go through the procedure.
We also welcome any advice/commitments that may improve the code quality, performance, etc.

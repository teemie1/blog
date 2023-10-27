## Create Image File From Flashdrive
~~~
$ sudo -i
$ dd if=/dev/sde bs=1G   status=progress | gzip > /mnt/data/images/raspiblitz-amd64-v1.10.0-2023-10-24-32G-ubuntu.img.gz
~~~

## Flash flashdrive from image file.
~~~
$ sudo -i
$ zcat /mnt/data/images/raspiblitz-amd64-v1.10.0-2023-10-24-32G-ubuntu.img.gz > /dev/sde
~~~

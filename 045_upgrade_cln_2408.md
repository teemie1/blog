# Upgrade Core Lightning Software 24.08

~~~
sudo systemctl stop lightningd
cd /tmp
wget https://github.com/ElementsProject/lightning/releases/download/v24.08/clightning-v24.08-Ubuntu-24.04.tar.xz
wget https://github.com/ElementsProject/lightning/releases/download/v24.08/SHA256SUMS
wget https://github.com/ElementsProject/lightning/releases/download/v24.08/SHA256SUMS.asc
sha256sum --ignore-missing --check SHA256SUMS
gpg --verify SHA256SUMS.asc
cd /
sudo tar -xvf /tmp/clightning-v24.08-Ubuntu-24.04.tar.xz
systemctl start lightningd

~~~

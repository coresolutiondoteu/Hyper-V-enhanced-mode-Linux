Credits
# https://c-nergy.be/blog/?p=13655
# https://askubuntu.com/questions/844245/how-to-compile-latest-pulseaudio-with-webrtc-in-ubuntu-16-04
# https://askubuntu.com/questions/496549/error-you-must-put-some-source-uris-in-your-sources-list
# https://unix.stackexchange.com/questions/65167/enable-udev-and-speex-support-for-pulseaudio
# https://rudd-o.com/linux-and-free-software/how-to-make-pulseaudio-run-once-at-boot-for-all-your-users
# https://gist.github.com/rkttu/35ecab5604c9ddc356b0af4644d5a226

# Installation and Enhanced session
# follow steps on the post below, I installed Ubuntu 22.04 on a Windows 11 machine 
# https://techbloggingfool.com/2020/12/26/deploy-a-linux-vm-on-hyper-v-with-sound-20-04-edition/
# or do these steps below

# Configure ubuntu for enhanced session
# Note - if it asks to reboot and rerun, rerun the install.sh again

wget https://raw.githubusercontent.com/Hinara/linux-vm-tools/ubuntu20-04/ubuntu/20.04/install.sh
chmod +x install.sh
sudo ./install.sh

# Back to windows powershell

Hyper-V\Set-VM -VMName "Your VM Name" -EnhancedSessionTransportType HvSocket

# Audio with some updates
# Install Some PreReqs
sudo apt-get install git libpulse-dev autoconf m4 intltool build-essential dpkg-dev libtool libsndfile1-dev libspeexdsp-dev libudev-dev -y

# Enable source code as downloadable.(can also do this from the gui Settings > About > Software & Updates)

sudo cp /etc/apt/sources.list /etc/apt/sources.list~
sudo sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list
sudo apt-get update

#  Pulseaudio. Download pulseaudio source in /tmp directory - Do not forget to enable source repositories

sudo apt build-dep pulseaudio -y
cd /tmp
apt source pulseaudio

# Compile pulseaudio
# go to the pulseaudio folder (pulseaudio-<version.number>) and build it from source, 
# what I did is follow what's currently in the README file, on the section HACKING - currently the instructions are these

cd pulseaudio-<version.number>
meson build
meson compile -C build
build/src/daemon/pulseaudio -n -F build/src/daemon/default.pa -p $(pwd)/build/src/

# Create xrdp sound modules
# go back to /tmp folder then clone

cd .. 
git clone https://github.com/neutrinolabs/pulseaudio-module-xrdp.git
cd pulseaudio-module-xrdp

# like the pulseaudio source, I followed their docs to get this done
# https://github.com/neutrinolabs/pulseaudio-module-xrdp/wiki/Build-on-Debian-or-Ubuntu

scripts/install_pulseaudio_sources_apt_wrapper.sh
./bootstrap
./configure PULSE_DIR=~/pulseaudio.src
sudo make install

# Restart Ubuntu, then the VM, hopefully it will all work for you as well. If it works...backup that VM, create a checkpoint :D
# Note. I did not use autologin in Ubuntu, I think autologin could cause issues with regards to sound. 
# I kinda guessed that from reading here https://github.com/neutrinolabs/pulseaudio-module-xrdp/wiki/README#install


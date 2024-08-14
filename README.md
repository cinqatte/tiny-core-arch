# Building a Minimal Linux System

---

This guide will take you step-by-step through the process of building a minimal Linux system using Arch Linux's `pacman` package manager as the foundation. Please ensure you have `sudo` privileges, as they are required for most of the tasks.

---

## 1. Set Up Your Working Directory

Start by setting an environment variable to specify where your new filesystem will be built:

```bash
export WORKDIR=$HOME/workdir
```

Now, create the working directory and subdirectories for your new operating system:

```bash
mkdir -p $WORKDIR && cd $WORKDIR
mkdir -p etc/pacman.d var/lib/pacman
```

---

## 2. Configure Network Resolution

For `pacman` to function, you need to set up domain name resolution. Create a `resolv.conf` file with your preferred DNS servers. The default configuration uses Cloudflare's DNS, but you can easily switch to Google, OpenDNS, or others by uncommenting the respective lines:

```bash
cat > etc/resolv.conf << "EOF"
# Cloudflare (Default)
nameserver 1.1.1.1
nameserver 1.0.0.1
nameserver 2606:4700:4700::1111
nameserver 2606:4700:4700::1001

# Google
#nameserver 8.8.8.8
#nameserver 8.8.4.4
#nameserver 2001:4860:4860::8888
#nameserver 2001:4860:4860::8844

# OpenDNS
#nameserver 208.67.222.222
#nameserver 208.67.220.220
#nameserver 2620:119:35::35
#nameserver 2620:119:53::53

# Others (uncomment as needed)
#nameserver 9.9.9.9
#nameserver 94.140.14.14
EOF
```

---

## 3. Set Up Pacman Configuration

Create a `pacman.conf` file to inform `pacman` where to download packages. Initially, signature checks are disabled to avoid potential GPG errors. These can be enabled later once the system is set up:

```bash
cat > etc/pacman.conf << "EOF"
[options]
Architecture = auto

[core]
SigLevel = Never 
Include = /etc/pacman.d/mirrorlist

[extra]
SigLevel = Never
Include = /etc/pacman.d/mirrorlist

[community]
SigLevel = Never
Include = /etc/pacman.d/mirrorlist
EOF
```

---

## 4. Select Package Mirrors

Download a list of package mirrors tailored to your region. The following example uses South African mirrors. You can adjust this to your location by fetching the relevant mirrorlist from [Arch Linux's mirrorlist](https://archlinux.org/mirrorlist/):

```bash
cat > etc/pacman.d/mirrorlist << "EOF"
## South Africa
Server = http://archlinux.za.mirror.allworldit.com/archlinux/$repo/os/$arch
Server = https://archlinux.za.mirror.allworldit.com/archlinux/$repo/os/$arch
Server = http://za.mirror.archlinux-br.org/$repo/os/$arch
Server = http://za.mirrors.cicku.me/archlinux/$repo/os/$arch
Server = https://za.mirrors.cicku.me/archlinux/$repo/os/$arch
Server = http://mirror.is.co.za/mirror/archlinux.org/$repo/os/$arch
Server = http://mirrors.urbanwave.co.za/archlinux/$repo/os/$arch
Server = https://mirrors.urbanwave.co.za/archlinux/$repo/os/$arch
EOF
```

---

## 5. Install Essential Tools

To avoid the complexity of building everything from scratch, download static versions of `pacman` and `bash` that can operate in your minimal environment:

```bash
sudo wget https://pkgbuild.com/~morganamilo/pacman-static/x86_64/pacman-static-6.0.1-2-x86_64.pkg.tar.xz
sudo tar xJf pacman-static-6.0.1-2-x86_64.pkg.tar.xz
sudo rm pacman-static-6.0.1-2-x86_64.pkg.tar.xz
sudo mv usr/bin/pacman-static usr/bin/pacman
sudo mv usr/bin/pacman-conf-static usr/bin/pacman-conf
sudo mv usr/bin/testpkg-static usr/bin/testpkg
sudo mv usr/bin/vercmp-static usr/bin/vercmp

sudo wget https://github.com/robxu9/bash-static/releases/download/5.2.015-1.2.3-2/bash-linux-x86_64
sudo mv bash-linux-x86_64 usr/bin/bash
sudo chmod +x usr/bin/bash
ln -s usr/bin bin
```

**Note:** This guide focuses on the `x86_64` architecture.

---

## 6. Mount Essential Filesystems

Mount the necessary virtual filesystems to your working directory:

```bash
mkdir -p $WORKDIR/{dev,proc,sys,run}
sudo mount --bind /dev $WORKDIR/dev
sudo mount -t devpts devpts -o gid=5,mode=0620 $WORKDIR/dev/pts
sudo mount -t proc proc $WORKDIR/proc
sudo mount -t sysfs sysfs $WORKDIR/sys
sudo mount -t tmpfs tmpfs $WORKDIR/run

if [ -h $WORKDIR/dev/shm ]; then
    sudo install -v -d -m 1777 $WORKDIR$(realpath /dev/shm)
else
    sudo mount -vt tmpfs -o nosuid,nodev tmpfs $WORKDIR/dev/shm
fi
```

---

## 7. Enter the Chroot Environment

Now that everything is set up, enter the chroot environment:

```bash
sudo chroot $WORKDIR
```

---

## 8. Initial Package Installation

To install the initial packages, follow these steps:

1. **Update the Package List:**

   In the chroot environment, refresh the `pacman` package list:

    ```bash
    pacman -Sy
    ```

2. **Clean Up Existing Binaries:**

   Open a second terminal, navigate to your `$WORKDIR`, and delete the existing `bin/bash` and `bin` directories to prevent conflicts:

    ```bash
    cd ${WORKDIR:?}
    sudo rm -rf bin/bash bin
    ```

3. **Install Essential Packages:**

   Return to the chroot environment and install essential packages, starting with `bash`:

    ```bash
    pacman -S bash
    ```

Now, you're ready to install additional packages and continue building your minimal Linux system.

---

# License

This project is licensed under the MIT License - see the LICENSE file for details.

---

# Contributing

Contributions are welcome! Feel free to open an issue or submit a pull request.
# ARM64 / AWS Graviton Build Instructions for ExtraHop rpcapd

This repository is a fork of ExtraHop’s `rpcapd` agent with additional patches required to successfully build and run it on **ARM64-based Linux systems**, including:

- AWS Graviton (Amazon Linux 2 / Amazon Linux 2023)
- Ubuntu ARM64
- Debian ARM64
- Other AArch64 Linux distributions

The original upstream project was based on very old WinPcap/libpcap code (circa ~2003), which does not compile on modern ARM systems without significant patching. This fork includes all fixes required to produce a functional, stable ARM64 `rpcapd` binary compatible with ExtraHop sensors.

## ✔ Summary of Changes in This Fork

### libpcap fixes
- Added missing Linux header: `#include <linux/sockios.h>` for `SIOCGSTAMP`
- Enabled remote capture support (`HAVE_REMOTE`)
- Updated Makefile to ensure bundled libpcap is used
- Added `<stdint.h>` for `UINT16_MAX`
- Removed static linking (`-static`) to avoid glibc issues on ARM
- Fixed configuration script compatibility for AArch64 (`--build=aarch64-unknown-linux-gnu`)

### rpcapd code fixes
- Removed x86-specific inline ASM (`sfence`, `lfence`)
- Replaced memory barriers with portable `__sync_synchronize()`
- Resolved multiple-definition error for `sockmain`
- Ensured rpcapd links against ARM-compatible libpcap and libcrypt

## ✔ Prerequisites

### Amazon Linux 2
```
sudo yum groupinstall -y "Development Tools"
sudo yum install -y libpcap libpcap-devel libxcrypt libxcrypt-devel flex bison git
```

### Amazon Linux 2023
```
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y libpcap libpcap-devel libxcrypt libxcrypt-devel flex bison git
```

### Ubuntu (ARM64)
```
sudo apt update
sudo apt install -y build-essential libpcap-dev libxcrypt-dev flex bison git autoconf automake libtool
```

## ✔ Clone This Fork

```
cd /opt
git clone https://github.com/0xM47H3W/rpcapd.git
cd rpcapd
```

## ✔ Build the Bundled libpcap

```
cd winpcap/wpcap/libpcap
./configure --build=aarch64-unknown-linux-gnu --enable-remote
make
```

## ✔ Build rpcapd

```
cd rpcapd
make clean || true
make CFLAGS="-g -O2 -Wno-error -DHAVE_REMOTE -I../"
```

Output binary:

```
winpcap/wpcap/libpcap/rpcapd/rpcapd
```

## ✔ Install rpcapd

```
sudo cp rpcapd /usr/local/sbin/rpcapd
sudo chmod 755 /usr/local/sbin/rpcapd
sudo chown root:root /usr/local/sbin/rpcapd
```

## ✔ Test rpcapd

Foreground:
```
sudo /usr/local/sbin/rpcapd -n -v
```

Passive:
```
sudo /usr/local/sbin/rpcapd -n -v -p 2002
```

Active with local listener:
```
nc -l 2003
sudo /usr/local/sbin/rpcapd -n -v -a 127.0.0.1,2003
```

## ✔ Run Against an ExtraHop Sensor

Active:
```
sudo /usr/local/sbin/rpcapd -n -v -a <sensor-ip>,2003
```

Passive:
```
sudo /usr/local/sbin/rpcapd -n -v -p 2002
```

## ✔ systemd Service

Create `/etc/systemd/system/rpcapd.service`:
```
[Unit]
Description=ExtraHop rpcapd ARM64 Agent
After=network-online.target

[Service]
ExecStart=/usr/local/sbin/rpcapd -a <sensor-ip>,2003 -n
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable:
```
sudo systemctl daemon-reload
sudo systemctl enable rpcapd
sudo systemctl start rpcapd
```

## ✔ Notes

- This fork is **not** an official ExtraHop release.
- ARM64 patches included for compatibility.
- Precompiled binaries may be included in GitHub Releases.


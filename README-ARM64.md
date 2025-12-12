# ARM64 / Graviton Build Instructions for rpcapd

This fork includes the changes required to build the ExtraHop rpcapd agent on ARM64 platforms such as:

- AWS Graviton (Amazon Linux 2 / 2023)
- Other modern ARM64 Linux distributions

## Changes in this fork (high level)

- Enabled remote capture support in libpcap (HAVE_REMOTE).
- Patched old libpcap sources for modern Linux headers (e.g. SIOCGSTAMP requires <linux/sockios.h>).
- Removed x86-only inline assembly (sfence / lfence) and replaced with portable memory barriers.
- Fixed missing includes for types like UINT16_MAX.
- Resolved multiple-definition of `sockmain` between rpcapd.c and pcap-new.c.
- Switched from static linking (-static) to dynamic linking to avoid missing -lc / -lcrypt issues.
- Ensured rpcapd links against libpcap and libcrypt on Amazon Linux.

## Build Steps (Amazon Linux / Graviton)

```bash
sudo yum groupinstall -y "Development Tools"
sudo yum install -y libpcap libpcap-devel libxcrypt libxcrypt-devel flex bison

cd /opt
sudo git clone https://github.com/matt-0123/rpcapd.git
sudo chown -R "$USER":"$USER" rpcapd
cd rpcapd/winpcap/wpcap/libpcap

# Enable remote capture support
./configure --build=aarch64-unknown-linux-gnu --enable-remote
make

# Build rpcapd
cd rpcapd
make clean || true
make
The resulting binary will be:

bash
Copy code
winpcap/wpcap/libpcap/rpcapd/rpcapd
Install it with:

bash
Copy code
sudo cp rpcapd /usr/local/sbin/rpcapd
sudo chmod 755 /usr/local/sbin/rpcapd
sudo chown root:root /usr/local/sbin/rpcapd
Run in active mode:

bash
Copy code
sudo /usr/local/sbin/rpcapd -n -v -a <sensor-ip>,2003

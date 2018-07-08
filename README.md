# shadowsocks-libev-dsm
 Synology DSM packages for Shadowsocks-libev.
 
# Installation
Download the spk for your architecture from the [Release](https://github.com/davidcava/shadowsocks-libev-dsm/releases) section, then install using Synology Package Center button _Manual Install_. See  architectures on [Synology knowledge base](https://www.synology.com/en-us/knowledgebase/DSM/tutorial/General/What_kind_of_CPU_does_my_NAS_have).

# Usage
After installing the package, create the configuration file(s) into `/var/packages/shadowsocks-libev/etc`.
Name must be: `ss-local.json` `ss-tunnel.json` `ss-redir.json` `ss-server.json`

If ss-redir is used, then routing will be activated and the traffic will be routed to ss-redir through iptables. udp will/might not work (see limitation below).

Additional instances can be started by creating more configuration files: `ss-local-xxx.json` `ss-tunnel-xxx.json` etc.

# Limitation
- Only works on DSM 6.1 and 6.2
- Configuration needs to manually edit the json configuration files through ssh, no graphical interface.
- DSM does not include the needed kernel modules for TProxy (at least on my model), which prevents using ss-redir with udp. Workaround is possible by recompiling the missing modules and iptables.
- When ss-redir is used, only incoming traffic is redirected (chain PREROUTING). DSM traffic itself is not sent to ss-redir (chain OUTPUT).
- obfs-plugin is not included for now
- I am using ss-local, ss-tunnel and ss-redir on my DS214play (evansport architecture) under DSM 6.2, anything else is not tested. Feedback welcome!

# Advanced
- It is possible to control which traffic goes to which ss-redir-xxx by putting iptables directives into files `ss-redir-xxx.ipt-rules-exclude` and/or `ss-redir-xxx.ipt-rules-include`. Packets matching the iptables rules within those files are excluded and/or included from being redirected.

Example: `/var/packages/shadowsocks-libev/etc/ss-redir-Korea.ipt-rules-exclude`
```sh
-m geoip --dst-cc EU,FR,LU,IT,PT,ES,DK,IE,GB,CH,SE,NL,AT,BE,DE
-m geoip --dst-cc AD,BG,CY,GI,GR,HU,IS,PL,FI,MT,NO,MC,RS,IM,CZ
```
# Build from source
- Setup the DSM toolkit for your model according to the official Synology [Developer's guide](https://developer.synology.com/developer-guide/)
- Download shadowsocks-libev source code into the toolkit
  ```sh
  cd /toolkit/source
  git clone https://github.com/shadowsocks/shadowsocks-libev.git
  cd shadowsocks-libev
  git submodule update --init --recursive
  ```
- Add or link the 2 subfolders `synology` and `SynoBuildConf` from `shadowsocks-libev-dsm` into the shadowsocks-libev source folder
  ```sh
  git clone https://github.com/shadowsocks/shadowsocks-libev-dsm.git
  ln -s ./shadowsocks-libev-dsm/SynoBuildConf ./shadowsocks-libev-dsm/synology .
  ```
- Download libev source into the toolkit and add the DSM build script
  ```sh
  cd /toolkit/source
  cvs -z3 -d :pserver:anonymous@cvs.schmorp.de/schmorpforge co libev
  cd libev
  ln -s ../shadowsocks-libev/shadowsocks-libev-dsm/libev/SynoBuildConf .
  ```
- Download mbedtls source into the toolkit and add the DSM build script
  (cannot use Synology provided mbedtls library because it misses MBEDTLS_CIPHER_MODE_CFB)
  ```sh
  cd /toolkit/source
  git clone https://github.com/ARMmbed/mbedtls
  cd mbedtls
  ln -s ../shadowsocks-libev/shadowsocks-libev-dsm/mbedtls/SynoBuildConf .
  ```
- Build for your architecture, example
  ```sh
  /toolkit/pkgscripts-ng/PkgCreate.py -p evansport -x0 -c shadowsocks-libev
  ```
- If everything went fine, package is now in `/toolkit/result_spk`

# Licence
    Copyright (c) 2018 David Cavallini

    shadowsocks-libev-dsm is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.
    
    shadowsocks-libev-dsm is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with shadowsocks-libev-dsm.  If not, see <http://www.gnu.org/licenses/>.

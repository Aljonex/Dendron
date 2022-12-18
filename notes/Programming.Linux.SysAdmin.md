---
id: 1z40ed9imtncu6183ysyvid
title: SysAdmin
desc: ''
updated: 1671401499996
created: 1671401303231
---
Finding how much storage is taken up `df -H` is in human-readable format.<br>
`sudo find / -xdev -type f -size +100M` finding files on whole current system with size of 100MB or bigger. <br>
`dpkg-query -Wf '${Installed-Size}\t${Package}\n' | sort -n | tail -n 30` shows 30 biggest packages on system.

### Getting rid of big packages
`sudo apt purge <package>` uninstalls and removes config files.<br>
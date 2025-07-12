---
{"dg-publish":true,"permalink":"/how-do-i-have-473-gb-of-used-disk-space/","created":"2025-07-12T06:30:34.981-07:00","updated":"2025-07-12T07:37:45.521-07:00"}
---

I was trying to run some tests today and out of nowhere [[Goodbye Docker Desktop, hello OrbStack\|OrbStack]] crashes so all my Docker containers are no longer reachable. 

I see OrbStack bouncing in the dock. After trying to start containers up again, I get this lovely error:

```
panic: rename /Users/nickdirienzo/.orbstack/log/vmgr.log /Users/nickdirienzo/.orbstack/log/vmgr.1.log: no space left on device

goroutine 17 [running, locked to thread]:
github.com/orbstack/macvirt/vmgr.check(...)
	github.com/orbstack/macvirt/vmgr/main.go:127
github.com/orbstack/macvirt/vmgr.runSpawnDaemon()
	github.com/orbstack/macvirt/vmgr/daemon.go:114 +0x46c
github.com/orbstack/macvirt/vmgr.Main()
	github.com/orbstack/macvirt/vmgr/main.go:1208 +0xf4
github.com/orbstack/macvirt/vmgr/cgostub/mainfunc.main(...)
	github.com/orbstack/macvirt/vmgr/cgostub/mainfunc/mainfunc.go:13
```

Excuse me? I have nearly half a TB in used disk and this laptop is only used for work. Where did all of this disk space go? Is this real?

```
nickdirienzo@Nicks-MacBook-Pro mirage % df -h
Filesystem        Size    Used   Avail Capacity iused ifree %iused  Mounted on
/dev/disk3s1s1   460Gi    10Gi   132Mi    99%    426k  1.4M   24%   /
devfs            205Ki   205Ki     0Bi   100%     710     0  100%   /dev
/dev/disk3s6     460Gi    11Gi   132Mi    99%      12  1.4M    0%   /System/Volumes/VM
/dev/disk3s2     460Gi   6.7Gi   132Mi    99%    1.2k  1.4M    0%   /System/Volumes/Preboot
/dev/disk3s4     460Gi   3.5Mi   132Mi     3%      58  1.4M    0%   /System/Volumes/Update
/dev/disk1s2     500Mi   6.0Mi   481Mi     2%       1  4.9M    0%   /System/Volumes/xarts
/dev/disk1s1     500Mi   5.4Mi   481Mi     2%      32  4.9M    0%   /System/Volumes/iSCPreboot
/dev/disk1s3     500Mi   2.4Mi   481Mi     1%      93  4.9M    0%   /System/Volumes/Hardware
/dev/disk3s5     460Gi   431Gi   132Mi   100%    5.5M  1.4M   80%   /System/Volumes/Data
map auto_home      0Bi     0Bi     0Bi   100%       0     0     -   /System/Volumes/Data/home
/dev/disk3s7     460Gi   2.0Mi   132Mi     2%     340  1.4M    0%   /Volumes/0CDCC59F-29F8-4E20-917C-C1A088D7A33D
```

Turns out, yes this is very real. I haven't had to diagnose this issue on Mac before. On Windows, there's WinDirStat which is quite helpful. o3 pointed me to `ncdu` which is effectively that but for the terminal. TIL! Sadly, it doesn't run in Ghostty.

I ran using `ncdu -x` which tells it to stay on the same filesystem (i.e. ignore Time Machine backups). After quickly scanning everything, it provides a helpful interactive overview:![Pasted image 20250712063631.png](/img/user/Pasted%20image%2020250712063631.png)
The overall usage numbers at the bottom are way over my actual hardware disk so not sure what's going on there. But I have 2 obvious places to dive into: `/System` and `/User`. Let's start with `/User`.

My home directory is using 329GB. My `code` directory which is where I keep everything checked out is 84GB! I want to blame node modules.

But `~/Library` seems to be the bigger culprit with 151GB used. 83GB are used by `~/Library/Containers` and nearly all of that is used by `~/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw`. 

I just migrated from Docker Desktop to OrbStack this week and OrbStack manages its own data elsewhere, so there's probably a duplicate 83GB elsewhere (narrator: there wasn't).

OrbStack stores its data in `~/Library/Group Containers/HUAQ24HBR6.dev.orbstack/data` per the [FAQ](https://docs.orbstack.dev/faq#orbstack-folder). And that also explains some of the apparent 16TB size because it has a 8TB sparse file for storing container assets. Is this also 83GB?

```
nickdirienzo@Nicks-MacBook-Pro data % du -h ~/Library/Group\ Containers/HUAQ24HBR6.dev.orbstack/data/data.img
5.5G    /Users/nickdirienzo/Library/Group Containers/HUAQ24HBR6.dev.orbstack/data/data.img
```

Nope. I'm glad I was wrong earlier about a duplicate 83GB. Because OrbStack manages its own data, [it doesn't need the `Docker.raw`](https://github.com/orgs/orbstack/discussions/1606) for managing Docker container images, volumes, etc so there's a wonderful reclaim of 83GB. I thought I removed Docker Desktop via `brew`? I guess it leaves the data around. That's good to know.

I can't fully delete the full `com.docker.docker` directory even with `sudo`. Whatever. At least I have 83GB back. I still have 248GB of data in my home directory, what now? 79GB in my `~/code/mirage` directory for everything related to that. Ah, 52GB for an unused dataset. Nice. Other fun finds are 1GB in Terraform providers for a single service -- I didn't realize it could be that big. 

Now we're at 135GB reclaimed. That's enough of a win. And no, it wasn't node's fault.
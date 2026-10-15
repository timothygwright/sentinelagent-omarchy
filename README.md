# SentinelOne Linux agent on Arch/Omarchy

This is a community packaging recipe for SentinelOne's proprietary Linux agent, not a supported SentinelOne or Omarchy release. Obtain the x86-64 `.deb` from your own SentinelOne management console; the binary and enrollment token must never be committed or uploaded to AUR.

**Tested in a disposable Omarchy VM:** Agent 26.2.1.24 (`pkgrel=4`) installed and started on `7.2.8-5-omarchy-bore`. After enrollment and several minutes of initialization, `sentinelctl control status` reported **Agent state Enabled** and **Agent health Healthy**, and `sentinelctl management status` reported **Connectivity On**. This verifies that this specific combination can start and report healthy in a VM; it does not establish vendor support, guarantee every feature works, or certify future rolling-kernel updates. The test VM was reset to an unenrolled snapshot after testing; the host agent is not installed.

The agent also logged `libbpf: map 's1_erb': failed to create: -EINVAL` and a warning about an unsupported probe path. Tracing showed requests for this ring buffer with `max_entries=0` failing, while requests with `max_entries=67108864` succeeded on the **same** kernel. A separately loaded vendor eBPF HTTP probe also worked. These warnings are not evidence for a kernel patch; investigate any affected feature with SentinelOne if needed.

Repeat installation, health checks, and rollback in a disposable VM booted with the actual Omarchy kernel for future releases. A Docker container shares the host kernel; the privileges needed for agent/eBPF testing would make it a poor substitute for VM isolation.

## Build and install

Place `SentinelAgent_linux_x86_64_v26_2_1_24.deb` in the build directory (or set `SRCDEST` to the directory containing it), inspect `PKGBUILD` and `sentinelagent.install`, then run:

```sh
makepkg -si
```

The package installs the systemd unit and creates an unprivileged `sentinelone` system account. It does not start or enable the service until you enroll it. The installer is verified against the `sha256sums` in `PKGBUILD`.

If your organization distributes a different installer build or SHA-256 digest for the same version, update `sha256sums` locally after verifying the vendor download. Never substitute `SKIP` for verification or publish the vendor binary.

In a visible terminal, authenticate with `sudo` before entering the token. Paste the **entire registration token** from the console (the base64 string, not just the decoded `site_key`):

```sh
sudo -v
read -r -s -p 'SentinelOne registration token: ' s1_token; printf '\n'
sudo /opt/sentinelone/bin/sentinelctl management token set "$s1_token"
unset s1_token
sudo systemctl enable --now sentinelone.service
sudo /opt/sentinelone/bin/sentinelctl control status
sudo /opt/sentinelone/bin/sentinelctl management status
```

Check that `control status` reports **Agent state Enabled** and **Agent health Healthy**, and `management status` reports **Connectivity On**. Initialization and initial connection can take several minutes. A running systemd unit by itself is not proof that the agent is healthy. Use `sudo journalctl -u sentinelone.service` and the agent logs under `/opt/sentinelone/log` to diagnose startup. Omarchy's rolling kernel and eBPF configuration require testing with each update.

## Roll back

In a visible terminal:

```sh
sudo /opt/sentinelone/bin/sentinelctl control stop
systemctl is-active sentinelone.service  # must report inactive
sudo systemctl disable sentinelone.service
mount | grep /opt/sentinelone  # check for mounts before deleting state
sudo umount /opt/sentinelone/mount            # if mounted
sudo umount /opt/sentinelone/ebpfs/bpf_mount  # if mounted
sudo pacman -Rns sentinelagent
```

When a management policy enables tamper protection, `control stop` or removal can require the **organization's authorized passphrase or console action**. Get that from your SentinelOne administrator rather than deleting binaries while the agent is still running.

Pacman leaves runtime state and the `sentinelone` system account behind. After confirming the service is inactive and package removal succeeded, remove them if you want a full local cleanup:

```sh
sudo rm -rf -- /opt/sentinelone
sudo userdel sentinelone
```

Keep runtime state if you intend to reinstall or troubleshoot an enrolled agent. Never delete `/opt/sentinelone` while the service or any of its mounts are still active.

## Publishing

An AUR package named `sentinelagent` already exists. Offer improvements to its maintainer instead of uploading a duplicate. Publish only `PKGBUILD`, `sentinelagent.install`, `.SRCINFO`, and these instructions; never publish the vendor `.deb`, a built `.pkg.tar.zst`, registration tokens, logs, or locally generated state. Update `pkgver`, `sha256sums`, and `.SRCINFO` together for each vendor release.

# NSS packages for OpenWrt qualcommax

An OpenWrt package feed for Qualcomm NSS offload on IPQ807x and IPQ60xx.
It runs the NSS cores on top of OpenWrt's upstream `qca_edma` and `qca_ppe`
ethernet drivers instead of `qca-nss-dp` and `qca-ssdk`. IPQ60xx support is
new and lightly tested.

The OpenWrt tree that uses this feed is
[openwrt-nss-edma](https://github.com/JuliusBairaktaris/openwrt-nss-edma),
branches `nss-edma-rework` (Linux 6.18) and `nss-edma-7.3` (6.18 and the
7.3 testing kernel). They carry the kernel patches and the
`kmod-qca-ppe-nss` module that connects `qca-nss-drv` to `qca_edma`. The
packages build for Linux 6.18 and 7.3 only.
Prebuilt images for IPQ807x and IPQ60xx boards are on the
[Qualcommax_NSS_Builder releases](https://github.com/JuliusBairaktaris/Qualcommax_NSS_Builder/releases)
page.

## Packages

| Package | Source | Version |
|---|---|---|
| `qca-nss-drv` | [lklm/nss-drv](https://git.codelinaro.org/clo/qsdk/oss/lklm/nss-drv) | `win.nss.1.0.r39`, `705629d` |
| `qca-mcs` | [lklm/qca-mcs](https://git.codelinaro.org/clo/qsdk/oss/lklm/qca-mcs) | `NHSS.QSDK.14.0.r9`, `063a467` |
| `qca-nss-ecm` | [lklm/qca-nss-ecm](https://git.codelinaro.org/clo/qsdk/oss/lklm/qca-nss-ecm) | `win.nss.1.0.r39`, `7894b76` |
| `qca-nss-clients` | [lklm/nss-clients](https://git.codelinaro.org/clo/qsdk/oss/lklm/nss-clients) | `NHSS.QSDK.12.5.5`, `51be82d` |
| `nss-userspace-oss` | [nss-userspace](https://git.codelinaro.org/clo/qsdk/oss/nss-userspace) | `win.nss.1.0.r39`, `dc142b3` |
| `nss-firmware` | [qosmio/qca-sdk-nss-fw](https://github.com/qosmio/qca-sdk-nss-fw) | 12.5 release 210, or 11.4.0.5 release 6 |
| `sqm-scripts-nss` | local | `nss-edma.qos` queue setup for SQM |

`qca-nss-clients` has no 14.0 branch. Only the modules this stack uses are
built: PPPoE, VLAN and bridge managers, the NSS qdisc, ingress shaping
(IGS), netlink, mirror and the Wi-Fi mesh manager. `nss-userspace-oss`
builds `libnl-nss` and `nssinfo`.

The default firmware is 12.5-210. Select `NSS_FIRMWARE_VERSION_11_4` for
802.11s mesh offload; newer firmware does not support mesh interfaces.

The OpenWrt packaging comes from
[qosmio/nss-packages](https://github.com/qosmio/nss-packages).

## Runtime behaviour

No module loads at boot. NSS offload is enabled at runtime, and the ECM and
SQM scripts refuse to start until the NSS data plane is up. On IPQ807x, once
the NSS firmware runs it receives all wired traffic, so every physical port
is attached to the NSS data plane.

## Building

Add the feed to `feeds.conf`:

```
src-git nss https://github.com/JuliusBairaktaris/nss-packages.git;edma-nss
```

A typical selection:

```
CONFIG_PACKAGE_kmod-qca-nss-drv=y
CONFIG_PACKAGE_kmod-qca-nss-ecm=y
CONFIG_PACKAGE_kmod-qca-nss-drv-pppoe=y
CONFIG_PACKAGE_kmod-qca-nss-drv-qdisc=y
CONFIG_PACKAGE_kmod-qca-nss-drv-igs=y
CONFIG_PACKAGE_sqm-scripts-nss=y
CONFIG_NSS_FIRMWARE_VERSION_12_5=y
CONFIG_NSS_MEM_PROFILE_MEDIUM=y
```

Set the memory profile to the board's RAM: `HIGH` for 1 GB, `MEDIUM` for
512 MB, `LOW` for 256 MB. The default is `HIGH` on ipq807x and `MEDIUM` on
ipq60xx.

## Acknowledgements

- [qosmio/nss-packages](https://github.com/qosmio/nss-packages) for the
  packaging, the kernel compatibility patches and the firmware tarballs.
- [Christian Marangi (Ansuel)](https://github.com/Ansuel) for the upstream
  EDMA and PPE drivers.
- [Robert Marko (robimarko)](https://github.com/robimarko) for maintaining the
  OpenWrt qualcommax target.

## Support

This is a single-maintainer project. Donations go toward development and
test hardware:
[GitHub Sponsors](https://github.com/sponsors/JuliusBairaktaris) or
[PayPal](https://paypal.me/JuliusBairaktaris).

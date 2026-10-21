# NSS firmware for OpenWrt qualcommax

An OpenWrt package feed carrying the Qualcomm NSS firmware for IPQ807x and
IPQ60xx, for
[openwrt-ppe-nss](https://github.com/JuliusBairaktaris/openwrt-ppe-nss),
branch `ppe-nss`. There the NSS cores are driven by `nss-wifi-drv`, and
wired offload, multicast, bridging and SQM run through the kernel's own
interfaces (flowtable, switchdev, tc), so no vendor host packages are
needed.

The vendor host stack (`qca-nss-drv`, `qca-nss-ecm`, `qca-nss-clients`,
`nss-userspace-oss` and `sqm-scripts-nss`) stays on the `edma-nss` branch
for openwrt-nss-edma.

## Packages

| Package | Source | Version |
|---|---|---|
| `nss-firmware` | [qosmio/qca-sdk-nss-fw](https://github.com/qosmio/qca-sdk-nss-fw) | 12.5 release 210, or 11.4.0.5 release 6 |

`nss-firmware` pulls in the firmware of the subtarget being built. The
default firmware is 12.5-210. Select `NSS_FIRMWARE_VERSION_11_4` for
802.11s mesh offload; newer firmware does not support mesh interfaces.

## Building

Add the feed to `feeds.conf`:

```
src-git nss https://github.com/JuliusBairaktaris/nss-packages.git;ppe-nss
```

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

# Third-Party Notices

The `.ipk` files in this folder are unmodified binary packages built by the OpenWrt project for
OpenWrt 23.05 (`aarch64_cortex-a53`). They are redistributed here only so the router can be
installed without internet access. Each package keeps its own license; none of them is covered
by this project's MIT license.

| Package | Version | License | OpenWrt source package |
|---|---|---|---|
| bash | 5.2.15-1 | GPL-3.0-or-later | `feeds/packages/utils/bash` |
| coreutils | 9.3-1 | GPL-3.0-or-later | `feeds/packages/utils/coreutils` |
| coreutils-shuf | 9.3-1 | GPL-3.0-or-later | `feeds/packages/utils/coreutils` |
| libreadline8 | 8.2-1 | GPL-3.0-or-later | `package/libs/readline` |
| libncurses6 | 6.4-2 | MIT | `package/libs/ncurses` |
| libubox20230523 | 2023-05-23-75a3b870-1 | ISC | `package/libs/libubox` |
| libblobmsg-json20230523 | 2023-05-23-75a3b870-1 | ISC | `package/libs/libubox` |
| uqmi | 2022-10-20-c8c9f105-1 | GPL-2.0 | `package/network/utils/uqmi` |
| wwan | 2019-04-29-6 | GPL-2.0 | `package/network/utils/wwan` |

## Source code

You can get the complete corresponding source code for these packages from the OpenWrt project:

- Build recipes: <https://github.com/openwrt/openwrt/tree/openwrt-23.05/package> and
  <https://github.com/openwrt/packages/tree/openwrt-23.05>
- Upstream source tarballs used by the build: <https://sources.openwrt.org/>
- Binary package feeds: <https://downloads.openwrt.org/releases/>

The full license texts are at <https://spdx.org/licenses/> under the identifiers listed above.

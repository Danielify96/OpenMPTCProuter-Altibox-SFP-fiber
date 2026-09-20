# Selectable-port Altibox 1G BASE-BX patch for OpenMPTCProuter

This community reference package is derived from the working Altibox/Tsuhan
`THMPRS-3511-10A` test on a Qotom Q20352G9. Unlike the first test build, the
manual Clause 37 workaround is not hardcoded to physical SFP7 or PCI
`0000:0b:00.1`.

The supplied binary and IPK are still exact-kernel builds for:

- OpenMPTCProuter v0.63
- x86_64
- Linux 6.12.40
- Intel X550EM_A_SFP_N, PCI ID 8086:15c4

Do not use the supplied binary after a firmware or kernel update.

## How port selection works

The driver has a writable string parameter named `altibox_an37_bdf`. It is
empty by default, so the workaround is disabled until an administrator
chooses one eligible interface. The selector converts an interface name such
as `eth6` into its stable PCI address and stores an early module-load option.

Only the selected Intel 8086:15c4 PCI function can use the workaround, and
only while configuring a 1 Gbit link. Other ixgbe ports follow normal driver
behaviour.

## Install through LuCI

1. Keep local-console or other out-of-band access available.
2. Upload `packages/omr-altibox-bx-sfp-selectable_20260920-1_x86_64.ipk`
   through **System > Software > Upload Package**.
3. Connect with SSH and list eligible interfaces:

       altibox-sfp-select --list

4. Select the desired SFP interface, for example:

       altibox-sfp-select eth6

5. Confirm the selection:

       altibox-sfp-select --status

6. Reboot manually when ready. The package never reboots automatically.

After boot, check:

    altibox-sfp-select --status
    dmesg | grep -i 'BX AN37 selectable'
    ethtool eth6

Replace `eth6` with the interface shown on that router.

Disable the workaround with `altibox-sfp-select --disable`, followed by a
manual reboot. Removing the package restores the saved previous driver when a
backup is available.

## Deliberately excluded

This package does not set or clone a MAC address. It does not create VLAN 102
or alter WAN, DHCP, firewall, routing, EEPROM, or laser settings. Configure
the required MAC and VLAN manually.

## Source patches

The `patches` directory contains the reproducible source sequence. Apply the
numbered patches in order to a clean Linux 6.12.40 ixgbe source tree. Patch
`0004` reproduces the tested, fixed-SFP7 implementation; patch `0005` then
replaces the hardcoded PCI address with the disabled-by-default module
parameter.

For another OpenMPTCProuter release, rebuild all patches against that exact
kernel source, configuration, compiler, and ABI. Verify vermagic, module
metadata, retpoline/BTF sections, and the target controller before installing.

The manual AN37 workaround was removed upstream after causing problems with
some link partners. Keep the explicit single-port restriction; do not enable
it globally across all ixgbe devices.

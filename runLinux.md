# Install Cerebras SDK
Installing the Singularity container build of the Cerebras: https://sdk.cerebras.net/installation-guide

Requested SDK, got download link via email from Mathias Jacquelin <mathias.jacquelin@cerebras.net>

## Cerebras SDK 2.10.0 Access
Here is a link with access to the SDK 2.10.0 files:
- [link](https://d12zdz04.na2.hubspotlinks.com/Ctc/5E+113/d12ZDZ04/VW1FlV57n7RrW12C8Yl2dKnlCW9k7knf5NnpJsN4sL4cn4ZD0CW5BWr2F6lZ3pDN22-gShHG59_W6wsMsD9jp_HwW937bm83J90z2W1Tgnwp2dZyCqW57Pb8j12Scz0W3ydxl67BwpL0W67Mgnh2kCdwtW65kQqH13CTynW91pSTp4yR5B4Vt0S1Y7zP8YcW5X7H3h94XC_fW7H3ytr2bCWZ6W8nvltH6NjdwgW5P8kPb4w-4lqN7nt1J425nPnW85WsR_5MZ6jWVWbr3S2f-tY2VZy1fx2GkTDwW7J18J383mNlcW3ppLXp6JvqNjW7d6r3036RVZXW14kJ-f16pbCqN6cbLT-4QWjyN3Vf7CwQCTl4N1nj3XBW6x3JW28C-g16l_4GlW1r_vXx8BXSHRW4hnn2W5p1vbcW9079Pg3fB9wcW4618RW7dn3JCW14SVhP1Y2c5rW3gJKgC2drS12W7sKLcb52Y024VvMX8v5XcY19f4pdwS604)
- http://sdk.cerebras.net is the documentation website.

## Rosetta-based installation on Apple-silicon Mac
https://sdk.cerebras.net/installation-guide#apple-silicon-mac-installation

```shell
brew install lima
```

Install Rosetta: https://sdk.cerebras.net/installation-guide#rosetta-based-installation
by line 47 in [config.yml](limaLinuxVM_withRosetta_config.yml).

## Run Linux

```shell
# Create a VM named cs_sdk - cs_sdk is already created by other project, don't run
limactl start limaLinuxVM_withRosetta_config.yml --name cs_sdk

# Start shell inside VM
limactl shell cs_sdk
```



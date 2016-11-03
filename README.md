# Eduroam Problems with Network Managers

OpenSSL started using TLS v1.1 and v1.2 as default from a
[commit](https://git.ti.com/wilink8-wlan/hostap/commit/35efa2479ff19c3f13e69dc50d2708ce79a99beb?format=patch)
back in November 2014. Not all RADIUS servers (which are used for
authentication) can handle TLS v1.2, described
[here](https://community.jisc.ac.uk/groups/eduroam/article/tls-12-and-updated-radius-requirements).

The problem can be avoided by setting the right flags, but most network managers
(at least `NetworkManager` and `wicd`) does not have a way to do this. (bug
report [here](https://bugzilla.gnome.org/show_bug.cgi?id=765059))

There has been a
[bugfix](https://git.ti.com/wilink8-wlan/hostap/commit/d4913c585ec9b62a667473878a7fd7d8600d3388?format=patch)
regarding the commit mentioned above, but this happened less than a month later
so is probably applied in the same release.

# Workaround

The workaround is to use the command line tools for eduroam, so you *can* set
the flags yourself.

This guide assumes you are running Ubuntu 16.04 (and that you are at the
University of Copenhagen).

You need to create a file that will contain the configuration for eduroam --
`eduroam.conf`:

```
ctrl_interface=/run/wpa_supplicant
network={
  ssid="eduroam"
  key_mgmt=WPA-EAP
  pairwise=CCMP
  group=CCMP TKIP
  eap=PEAP
  domain_suffix_match="radius.ku.dk"
  identity="<abc123>@ku.dk"
  password="YOUR-PASSWORD-AS-PLAINTEXT"
  phase1="tls_disable_tlsv1_1=1 tls_disable_tlsv1_2=1"
  phase2="auth=MSCHAPV2"
}
```

Then each time you want to connect, do the following (run `iwconfig` to find the
name of your wireless interface)

#### 1. Stop NetworkManager so you can control the network connections yourself.

``` shell
sudo systemctl stop NetworkManager.service
```

#### 2. Connect to the wireless network

``` shell
sudo wpa_supplicant -i <wireless-interface> -c eduroam.conf
```

#### 3. Set up DHCP, so you are assigned an IP address. (I'm using `dhclient`, but
   you can probably use something else like `dhcpd`)

``` shell
sudo dhclient <wireless-interface>
```

#### 4. If DHCP does not give you a DNS server (which my setup doesn't), you can edit
   the file `/etc/resolv.conf` by hand to make

``` shell
sed -e 's/127.0.1.1/8.8.8.8/' /etc/resolv.conf | sudo tee /etc/resolv.conf
```

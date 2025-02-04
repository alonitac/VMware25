# Common cluster issues and troubleshooting

## Not responding host

The root cause for vCenter being unable to communicate with your host it misconfigurations 
in IP and hostname. 

Your ESXi host should have a hostname that matches its assigned IP. For example:

- **IP Address**: 192.168.122.221
- **Expected Hostname**: ip-192-168-122-221

If your hostname does not match this format, ensure that your ESXi host is using dynamic IP allocation via DHCP.

How to Fix It:

1. Open the ESXi direct console.
2. Press **F2** to log in with root credentials.
3. Select **Configure Management Network** → **IPv4 Configuration**.
4. Select **Use dynamic IPv4 address and network configuration**.
5. Press **Enter** to close the configuration window. Press **Esc** to exit the menu while saving the new configurations. 
6. Shut-down the host, and start it again. The IPv4 and hostname should match.


## Host connection and power state

In case of **Host connection and power state** alarm, if your host IPv4 address and the hostname are matched (e.g. IP `192.168.122.12` and hostname `ip-192-168-122-12`), 
it's probably due to the fact that our lab is turned off, so all hosts and the vCenter. Try to reboot the host. 

> [!NOTE]
> To understand the meaning of this alarm, choose your host, go to **Configure** tab, and choose **Alarm Definition**. 
> Search the alarm and review the alarma rule definition. 
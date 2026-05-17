# RDP and SOCKS Tunneling with SocksOverRDP

In some assessments, we might be in a network that only has Windows hosts and may not be able to use SSH for pivoting. We can use [SocksOverRDP](https://github.com/nccgroup/SocksOverRDP) that uses **Dynamic Virtual Channels** from the Remote Desktop Service feature of Windows. DVC is responsible for tunnelling packets over the RDP connection. This feature can help in clipboard data transfer and audio sharing.

We'll use `SocksOverRDP` to tunnel our custom packets and then proxy through it. We will use the tool [Proxifier](https://www.proxifier.com/) as our proxy server.

We will need:
1. [SocksOverRDP x64 Binaries](https://github.com/nccgroup/SocksOverRDP/releases)
2. [Proxifier Portable Binary](https://www.proxifier.com/download/#win-tab)

From the Windows target, we will then need to load the `SocksOverRDP.dll` using `regsvr32.exe.`
#### Loading SocksOverRDP.dll using regsvr32.exe

```powershell
regsvr32.exe SocksOverRDP-Plugin.dll
```

![[SocksOverRDP.dll.png]]

Now we can connect to 172.16.5.19 (The internal host) over RDP using `mstsc.exe`, and we should receive a prompt that the `SocksOverRDP` plugin is enabled, and it will listen on 127.0.0.1:1080.

We will need to<span style="color:rgb(0, 176, 80)"> transfer SocksOverRDPx64.zip</span> or just the <span style="color:rgb(0, 176, 80)">SocksOverRDP-Server.exe</span> <span style="color:rgb(192, 0, 0)">to</span> 172.16.5.19 <span style="color:rgb(192, 0, 0)">(The internal host)</span>. We can then<span style="color:rgb(0, 176, 80)"> start SocksOverRDP-Server.exe with Admin privileges</span>.

When we go back to our foothold target and check with Netstat, we should see our SOCKS listener started on 127.0.0.1:1080.

After starting our listener, we can <span style="color:rgb(0, 176, 80)">transfer</span> `Proxifier` portable to the Windows 10 target (on the 10.129.x.x network / <span style="color:rgb(0, 176, 80)">The external network</span>), and configure it to forward all our packets to 127.0.0.1:1080
#### RDP Performance Considerations

If the RDP was very slow: We can access the `Experience` tab in mstsc.exe and set `Performance` to `Modem`.


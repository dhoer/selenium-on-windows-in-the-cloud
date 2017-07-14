# Running Selenium on Windows in the Cloud

I was tasked to create a windows selenium-grid over three years ago and I
still see people struggling with the same issues I ran into. There are great
SAAS providers like Sauce Labs, but sometimes you need to build your own for 
proprietary reasons.  Here is a list of gotchas that I had to work around.
 
Note that selenium-grid contains a hub (master) and nodes (slaves).  If you 
are building selenium as a standalone, then substitute where I mention node
with standalone.

## Gotcha #1 - GUI Service

While the selenium hub can run as a windows service, that is not the case
for a selenium node. Windows services run in [Session 0 Isolation](https://msdn.microsoft.com/en-us/library/windows/hardware/dn653293(v=vs.85).aspx). 
This means the service can't drive the Graphical User Interface.  So what do you do? 

You have to start the application from a command shell. To do this, create 
a command file e.g., selenium.cmd to start selenium node and place it the 
user's startup folder:

`C:\Users\<user name>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`.

If you use Chef configuration management, the [selenium](https://supermarket.chef.io/cookbooks/selenium) 
cookbook can do perform this step for you.

## Gotcha #2 - Auto Login

While windows services automatically start on server reboot, 
the user account won't. So how do you manage tahat?  By setting windows
registry to auto logon to that account e.g.,

```
[HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon]
"AutoAdminLogon"="1"
"DefaultUsername"="<user name>"
"DefaultPassword"="mypassword"
"DefaultDomainName"=""
```

If you use Chef configuration management, the [windows_autologin](https://supermarket.chef.io/cookbooks/windows_autologin) 
cookbook can perform this step for you.

## Gotcha #3 - Display Resolution

When running a windows server in the cloud, there is not a display device 
attached.  This is considered headless mode.  If you are on Amazon, the
default resolution of the display driver can be pretty low.
So how do you go about changing the default resolution? Well, create another user e.g.,
`rdp_local` and create a startup script to [RDP](https://msdn.microsoft.com/en-us/library/aa383015(v=vs.85).aspx) 
into other user account at resolution desired e.g., 

`cmdkey.exe /add:localhost /user:rdp_local /pass:"mypassword" | mstsc.exe /v localhost /w:1280 /h:1024`.

You will also have to set some registry settings to [enable RDP](https://technet.microsoft.com/en-us/library/cc722151%28v=ws.10%29.aspx) 
and suppress [remote computer cannot be verified prompt](http://www.mytecbits.com/microsoft/windows/rdp-identity-of-the-remote-computer)
e.g., 

```
[HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server]
"fDenyTSConnections"=dword:00000000

[HKLM\SOFTWARE\Microsoft\Terminal Server Client]
"AuthenticationLevelOverride"=dword:000000000
```

Finally, add RDP firewall rule: 

`netsh advfirewall firewall add rule name="RDP" protocol=TCP dir=in profile=public localport=3389 remoteip=localsubnet localip=any action=allow`

If you use Chef configuration management, the [windows_screenresolution](https://supermarket.chef.io/cookbooks/windows_screenresolution) 
cookbook can perform these steps for you.


---
I hope this information helps those struggling with windows.  If you have a 
better way of doing this, find something inaccurate,
or want to share your experience, then please share via issues.




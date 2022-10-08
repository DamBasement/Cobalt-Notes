# CobaltStrike startup
Two main components
- Server (on linux machine)
- Dashboard (on winzoz machine)

## Server
CobaltStrike server runs on linux machine, it can be started from folder `/opt/CobaltStrike/`
running the following command:
```
sudo ./teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/webbug.profile
```

where the IP is the linux machine IP where you want to listen to and the password will be used
with the CobaltStrike client to allow the connection
### Run as a Service
```
attacker@ubuntu ~> sudo vim /etc/systemd/system/teamserver.service
```
Then copy the following into that file
```
[Unit]
Description=Cobalt Strike Team Server
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=root
WorkingDirectory=/home/attacker/cobaltstrike
ExecStart=/home/attacker/cobaltstrike/teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/webbug.profile

[Install]
WantedBy=multi-user.target
```
type
```
sudo systemctl daemon-reload
```
and finally
```
sudo systemctl enable --now teamserver.service 
```

## Dashboard
CobaltStrike client Dashboard runs on winzoz server, can be started just clicking on the icon
It will ask for IP, port and pass we set on the server

# CobaltStrike usage
It is so important the `listener` view available in the `CobaltStrike` menu and
the running `Web Delivery` available on `attack -> WebDrive-by -> Manage` that
will show a list of listening Web Delivery

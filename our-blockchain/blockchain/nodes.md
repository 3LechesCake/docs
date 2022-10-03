# Nodes

Download the miner from [https://github.com/3LechesCake/Blockchain/tree/main/Miner](https://github.com/3LechesCake/Blockchain/tree/main/Miner)

once downloaded ensure is connected to the bootnode and you should be able to mine transactions for the blockchain and earn tokens.

Create Boot Node as Service

sudo nano /etc/systemd/system/bootnode.service

```
[Unit]
Description=Ethereum BNode
After=syslog.target network.target
Wants=network.target

[Service]
User=tresservice
Group=tresservice
Type=simple
ExecStart=bootnode -nodekey boot.key -verbosity 7 -addr 147.182.143.50:30301

KillMode=process
KillSignal=SIGINT
TimeoutStopSec=90
Restart=always
RestartSec=5s

[Install]
WantedBy=defaul.target
```

Create Node 1 as Service

sudo nano  **/etc/init.d/geth**



```
#!/bin/bash
#
### BEGIN INIT INFO
# Provides:          geth
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Should-Start:      $network $named $time
# Should-Stop:       $network $named $time
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start and stop the geth daemon
# Description:       Controls the geth server daemon
### END INIT INFO
#
. /lib/lsb/init-functions
start() {   
   PCOUNT=`pgrep -c geth`
   if ((PCOUNT==1)); then
      /usr/bin/bootnode -nodekey /usr/local/bnode/boot.key -verbosity 5 -addr 147.182.143.50:30301 -nat extip:147.182.143.50 &
   fi
   echo $"Bnode Started"
}
stop() {
   PCOUNT=`pgrep -c geth`
   if ((PCOUNT>1)); then   
      /usr/bin/killall -9 geth &
   fi
   echo "Geth Stopped"
}
case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  status)
        status geth
        ;;
  restart|reload|condrestart)
        stop
        start
        ;;
  *)
  echo $"Usage: $0 {start|stop|restart|reload|status}"
  exit 1
esac
exit 0
```



Set Permissions

```
sudo chmod 755 /etc/init.d/geth
sudo chown root:root /etc/init.d/geth
```

Enable Geth Service

<pre><code><strong>sudo update-rc.d geth defaults
</strong>sudo update-rc.d geth enable</code></pre>

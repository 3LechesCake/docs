# Nodes

Download the miner from [https://github.com/3LechesCake/Blockchain/tree/main/Miner](https://github.com/3LechesCake/Blockchain/tree/main/Miner)

once downloaded ensure is connected to the bootnode and you should be able to mine transactions for the blockchain and earn tokens.

Create bootnode --genkey=boot.key

then create the following service.

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
ExecStart=bootnode -nodekey /usr/local/testnet/bnode/boot.key -verbosity 7 -addr 147.182.143.50:30301

KillMode=process
KillSignal=SIGINT
TimeoutStopSec=90
Restart=always
RestartSec=5s

[Install]
WantedBy=defaul.target
```

sudo systemctl enable bnode

sudo systemctl start bnode



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
      /usr/bin/geth --networkid 6066 --datadir /home/blockservice/node1/data --identity "Main Node" --bootnodes enode://c8b59a5915b0df7f1bdaea6bd2f7c4e8056f3393f8198acdc09b80ab46f9b9913d1f28452c3da8a8d9c93983278ff546f22a9d7222c12b2200c9503f7144f4a8@147.182.143.50:0?discport=30301 --port 30304 --miner.threads 1 --miner.etherbase=0x4f00aa38162c6f24c505c9374c7b6b4631186066 --ipcdisable --syncmode full --http --rpc.allow-unprotected-txs --http.vhosts "rpc.tresleches.finance"  --http.corsdomain "*" --http.port 8545 --nodiscover --nat extip:147.182.143.50  --http.api admin,eth,miner,net,txpool,personal,web3 --http.addr 147.182.143.50 --ws --ws.addr 147.182.143.50 --ws.port 8549 --ws.api eth,net,web3  --miner.etherbase 0xe6D74C742698D26511504FBA2261a6097d556066  --ws.origins "rpc.tresleches.finance" --rpc.gascap 80000000  --rpc.txfeecap 2  &
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
</strong>sudo update-rc.d geth enable
</code></pre>

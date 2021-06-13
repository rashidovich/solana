# Running telegraf in docker container

There's a cool [guide](https://github.com/stakeconomy/solanamonitoring) about Solana node monitoring. It describes how one could setup monitoring using [Telegraf](https://github.com/influxdata/telegraf) + [InfluxDb](https://github.com/influxdata/influxdb) + [Grafana](https://github.com/grafana/grafana).

All these instruments could be wrapped into docker containers. I'll show how to spin-up Telegraf instance.

**Prerequisites**:

`Docker`

sudo apt install docker.io curl -y \
&& sudo systemctl start docker \
&& sudo systemctl enable docker

`Telegraf config`

If you plan to send metrics to Solana community dashboard then use [this config](https://github.com/stakeconomy/solanamonitoring#example-telegraf-configuration), if you create your own environment (influxDb + grafana) then you might use same config but with substituted values in [[outputs.influxdb]] section.
In both cases we update [[inputs.exec]] section with new **commands**

[[inputs.exec]]  
&ensp;commands = ["/root/solanamonitoring/monitor.sh -s /bin/bash root"]

store telegraf config using path: /home/user/telegraf/telegraf.conf

`Minotiring shell script`

Could be taken from [here](https://github.com/stakeconomy/solanamonitoring/blob/main/monitor.sh) (don't forget to update values accordingly)

identityPubkey="your pubkey"   
voteAccount="your voteaccount"  
rpcURL="your rpc url" #usually http://127.0.0.1:8899  

store monitor.sh using path: /home/user/solanamonitoring/monitor.sh

`Docker command`

sudo docker run -d \
--network="host" \
--name=telegraf \
--restart=always \
-v /home/user/telegraf/telegraf.conf:/etc/telegraf/telegraf.conf \
-v /home/user/solanamonitoring:/root/solanamonitoring \
-v /home/user/.config/solana:/root/.config/solana \
-v /home/user/.local/share/solana:/root/.local/share/solana \
-v /:/hostfs:ro \
-v /etc:/hostfs/etc:ro \
-v /proc:/hostfs/proc:ro \
-v /sys:/hostfs/sys:ro \
-v /var/run/utmp:/var/run/utmp:ro \
-e HOST_ETC=/hostfs/etc \
-e HOST_PROC=/hostfs/proc \
-e HOST_SYS=/hostfs/sys \
-e HOST_MOUNT_PREFIX=/hostfs \
telegraf:latest
# Running telegraf in docker container

There's a cool [guide](https://github.com/stakeconomy/solanamonitoring) about Solana node monitoring describes how one could setup monitoring using [Telegraf](https://github.com/influxdata/telegraf) + [InfluxDb](https://github.com/influxdata/influxdb) + [Grafana](https://github.com/grafana/grafana).

All instruments could be wrapped into docker containers. I'll show only how to spin-up Telegraf instance.

**Prerequisites**:

`Docker`

sudo apt install docker.io curl -y && \\
sudo systemctl enable docker && \\
sudo systemctl start docker

`Telegraf config`

If you plan to send metrics to Solana community dashboard then use [this config](https://github.com/stakeconomy/solanamonitoring#example-telegraf-configuration), if you create your own environment (influxDb + grafana) then you might use same config but with substituted values in [[outputs.influxdb]] section.

In both cases we update [[inputs.exec]] section with new **commands**

[[inputs.exec]]  
&ensp;commands = ["/root/solanamonitoring/monitor.sh -s /bin/bash root"]

save telegraf config on server using path: /home/*user*/telegraf/telegraf.conf

`Monitoring shell script`

Could be taken from [here](https://github.com/stakeconomy/solanamonitoring/blob/main/monitor.sh) (don't forget to update values accordingly)

*identityPubkey*="your pubkey"   
*voteAccount*="your voteaccount"  
*rpcURL*="http://127.0.0.1:8899"

save monitor.sh on server using path: /home/*user*/solanamonitoring/monitor.sh

`Docker command`

sudo docker run -d \
--network="host" \
--name=telegraf \
--restart=always \
-v /home/*user*/telegraf/telegraf.conf:/etc/telegraf/telegraf.conf \
-v /home/*user*/solanamonitoring:/home/*user*/solana/solanamonitoring \
-v /home/*user*/.config/solana:/home/*user*/.config/solana \
-v /home/*user*/.local/share/solana:/home/*user*/.local/share/solana \
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

`Configure telegraf container`

*Monitor.sh* has ***jq*** and ***bc*** dependancies, we need to install them inside of docker container, therefore

1. connect to container ***sudo docker exec -it telegraf bash***
2. run command ***apt-get update && apt-get install jq && apt-get install bc***
3. check that solana is connected to the right cluster (devnet, testnet, mainnet)
***/home/*user*/.local/share/solana/install/active_release/bin/solana config get***

> ![image](https://user-images.githubusercontent.com/5165742/121822575-1ee3a080-cca0-11eb-8944-717fdc6bed8b.png)
> 
> to connect to testnet cluster, use ***/home/*user*/.local/share/solana/install/active_release/bin/solana config set --url http://api.testnet.solana.com***

5. ***exit***
6. check that telegraf logs are Ok ***sudo docker logs telegraf -f --tail 20***

`Done`

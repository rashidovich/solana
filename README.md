# Running telegraf in docker container

There's a cool guid related to Solana node monitoring. It shows how one could setup monitoring using [Telegraf](https://github.com/influxdata/telegraf) + [InfluxDb](https://github.com/influxdata/influxdb) + [Grafana](https://github.com/grafana/grafana).

All these instruments could be wrapped into docker containers. I'll show how to spin-up Telegraf instance.

sudo docker run -d \
--network="host" \
--name=telegraf \
--restart=always \
-v /home/user/telegraf/telegraf.conf:/etc/telegraf/telegraf.conf \
-v /home/user/solanamonitoring:/home/user/solanamonitoring \
-v /home/user/.config/solana:/home/user/.config/solana \
-v /home/user/.local/share/solana:/home/user/.local/share/solana \
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

Identify the ports

```
./dpdk_setup_ports.py -s
```

Create a configuration file 

```
cp  cfg/simple_cfg.yaml  /etc/trex_cfg.yaml
```

trex_cfg.yaml format

```
- port_limit      : 2
  version         : 2
  interfaces    : ["03:00.0","03:00.1"] 
  port_info       :
          - ip         : 1.1.1.1
            default_gw : 2.2.2.2
          - ip         : 2.2.2.2
            default_gw : 1.1.1.1
  
################ or mac ####################
          - dest_mac : '00:00:00:05:00:00'
            src_mac  : '00:00:00:06:00:00' # new format
          - dest_mac : [0x0,0x0,0x0,0x7,0x0,0x01]
            src_mac  : [0x0,0x0,0x0,0x8,0x0,0x02] # old format

```

启动stateless服务器

```
./t-rex-64 -i
```

基本运行

```
[bash]>trex-console
trex>start -f stl/udp_1pkt_simple.py -m 10mbps -p 0       #直接运行python脚本
trex>service                                            #切换到service模式下
trex(service)>l2 -p 0 --dst 00:10:10:11:11:11       # L2模式 -p 端口号  --dst 目的MAC
trex(service)>l3 -p 0 --src 192.168.10.11 --dst 192.168.10.168     #L3模式
trex(service)>ping -p 0 -d 192.168.10.168          # -p 端口号  -d dest_ip
trex(service)>arp    #L3模式，端口获得的MAC
trex(service)>portattr    #端口信息
```

运行python

```
[bash]>trex-console                                                    #1

# Start the traffic on all ports at 10 mbps.
trex>start -f stl/udp_1pkt_simple.py -m 10mbps -p 0  #-m 后面值可为10 50%

# pause  the traffic on all port
>pause -a

# resume  the traffic on all port
>resume -a

# stop traffic on all port
>stop -a

# show dynamic statistic
>tui
```

udp_1pkt_simple.py文件：

```python
from trex_stl_lib.api import *

class STLS1(object):

    def create_stream (self):
        return STLStream( 
            packet = 
                    STLPktBuilder(
                        pkt = Ether()/IP(src="16.0.0.1",dst="48.0.0.1")/
                                UDP(dport=12,sport=1025)/(10*'x')
                    ),
             mode = STLTXCont())

    def get_streams (self, direction = 0, **kwargs):
        # create 1 stream 
        return [ self.create_stream() ]


# dynamic load - used for trex console or simulator
def register():
    return STLS1()
```


# monitor-cpumem-user

### How to monitor CPU or memory usage on a per-user basis

Using command "ps"

```bash
# ps axo user,pcpu,pmem,rss --no-heading | awk '{pCPU[$1]+=$2; pMEM[$1]+=$3; sRSS[$1]+=$4} END {for (user in pCPU) if (pCPU[user]>0 || sRSS[user]>10240) printf "%s:@%.1f%% of total CPU,@%.1f%% of total MEM@(%.2f GiB used)\n", user, pCPU[user], pMEM[user], sRSS[user]/1024/1024}' | column -ts@ | sort -rnk2

influxdb:  0.2% of total CPU,  30.5% of total MEM  (0.29 GiB used)
root:      0.0% of total CPU,  12.4% of total MEM  (0.13 GiB used)
postfix:   0.0% of total CPU,  1.2% of total MEM   (0.01 GiB used)
polkitd:   0.0% of total CPU,  1.6% of total MEM   (0.02 GiB used)
grafana:   0.0% of total CPU,  5.5% of total MEM   (0.05 GiB used)
```
Using command "top"

```bash
# name=utop
# mkdir -p /usr/local/etc/$name
# cat >/usr/local/etc/$name/.toprc <<\EOF
RCfile for "top with windows"       # shameless braggin'
Id:a, Mode_altscr=0, Mode_irixps=0, Delay_time=3.000, Curwin=0
Def fieldscur=aEhioqtwKNmbcdfgjplrsuvyzx
    winflags=34105, sortindx=10, maxtasks=0
    summclr=1, msgsclr=1, headclr=3, taskclr=1
Job fieldscur=ABcefgjlrstuvyzMKNHIWOPQDX
    winflags=62777, sortindx=0, maxtasks=0
    summclr=6, msgsclr=6, headclr=7, taskclr=6
Mem fieldscur=ANOPQRSTUVbcdefgjlmyzWHIKX
    winflags=62777, sortindx=13, maxtasks=0
    summclr=5, msgsclr=5, headclr=4, taskclr=5
Usr fieldscur=ABDECGfhijlopqrstuvyzMKNWX
    winflags=62777, sortindx=4, maxtasks=0
    summclr=3, msgsclr=3, headclr=2, taskclr=3
EOF
# echo "$name's .toprc config file is in this dir" >/usr/local/etc/$name/README
```
```bash
# cat >/usr/local/bin/$name <<EOF
#!/bin/bash
HOME=/usr/local/etc/$name top -bn1 | awk '
  NR>2 && NF>0 { pCPU[\$1]+=\$2 ; pMEM[\$1]+=\$3 }
  END {
    for (user in pCPU) if (pCPU[user]>0.5 || pMEM[user]>0.5)
      printf "%s:@%.1f%% of total CPU,@%.1f%% of total MEM\\n", user, pCPU[user], pMEM[user]
  }
' | column -ts@ | sort -rnk2
EOF
chmod +x /usr/local/bin/$name
```
```bash
# utop
root:      6.7% of total CPU,  14.2% of total MEM
postfix:   0.0% of total CPU,  1.3% of total MEM
polkitd:   0.0% of total CPU,  1.6% of total MEM
influxdb:  0.0% of total CPU,  30.5% of total MEM
grafana:   0.0% of total CPU,  5.6% of total MEM
```

Source : https://access.redhat.com/solutions/239483

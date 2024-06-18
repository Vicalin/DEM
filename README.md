CLI    CLI-ISP   CLI-HQR
ISP    ISP-HQR   ISP-BRR CLI-ISP
HQR    ISP-HQR   HQR-HQSRV CLI-HQR
HQSRV  HQR-HQSRV
BRR    ISP-BRR   BRR-BRSRV
BR-SRV BRR-BRSRV

cli.hq.work
33.33.33.2/24
44.44.44.2/30,44.44.44.1 hq.work 172.16.100.2

isp
11.11.0.1/24
22.22.0.1/24
33.33.33.1/24

hq-r
11.11.0.2/24
172.16.100.1/26
44.44.44.1/30,44.44.44.2

hq-srv.hq.work
dhcp

br-r
22.22.0.2/24
192.168.100.1/28

br-srv.branch.work
192.168.100.2/28,192.168.100.1 172.16.100.2 22.22.0.1 hq.work branch.work

HQR,ISP,
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding=1" >> /etc/sysctl.conf
10.10.10.0/30
nmcli connection modify tun1 ip-tunnel.ttl 64

nano /etc/frr/daemons
systemctl enable --now frr
vtysh
conf t
router ospf
passive-interface default
network 10.10.10.0/30 area 0
network LOC!!!NET area 0
exit
interface tun1
no ip ospf network broadcast
no ip ospf passive
exit
do write memory
exit

BRR
/etc/dhcp/dhcpd.conf

subnet 172.16.100.0 netmask 255.255.255.192 {
 range 172.16.100.2 172.16.100.12;
 option domain-name-servers 172.16.100.1;
 default-lease-time 600;
 max-lease-time 7200;
}
host hq-srv.hq.work {
 hardware ethernet mac;
 fixed-address 172.16.100.2;
}

systemctl enable --now dhcpd

BRR
useradd branch-admin -m -c "Branch admin" -U
passwd branch-admin
useradd network-admin -m -c "Network admin" -U
passwd network-admin
BRSRV
useradd branch-admin -m -c "Branch admin" -U
passwd branch-admin
useradd network-admin -m -c "Network admin" -U
passwd network-admin
HQR
useradd network-admin -m -c "Network admin" -U
passwd network-admin
useradd admin -m -c "Admin" -U
passwd admin
HQSRV
useradd admin -m -c "Admin" -U
passwd admin

ISP
iperf3 -s
HQR
iperf3 -c 11.11.0.1

mkdir /var/backup

data=$(date +%d.%m.%Y-%H:%M:%S)
mkdir /var/backup/$data

cp -r /etc/frr /var/backup/$data
cp -r /etc/nftables /var/backup/$data
cp -r /etc/NetworkManader/system-connections /var/backup/$data
cp -r /etc/dhcp /var/backup/$data

cd /var/backup

tar czfv "./$data.tar.gz" ./$data
rm -r /var/backup/$data

/etc/ssh/sshd_config
/etc/selinux/config permissive
setenforce 0
systemctl restart sshd

/etc/nftables/main.nft
table inet nat {
 chain prerouting {
  type net hook prerouting priority filter; policy accept;
  ip daddr 11.11.0.2 tcp dport 2222 dnat ip to 172.16.100.2:2222
 }
 chain my_masquerade {
  type nat hook postrouting priority srcnat;
 }
}
systemctl enable --now nftables.service

/etc/sysconfig/nftables.conf
table inet filter {
 chain input {
  type filter hook filter hook input priority filter;
  policy accept;
  ip saddr 33.33.33.0/24 tcp dport 2222 counter drop;
 }
}

/etc/chrony.conf
# pool pool.ntp.org iburst
HQR
server 172.0.0.1 iburst prefer
hwtimestamp *
local stratum 5
allow 0/0

reboot

# pool pool.ntp.org iburst
server 11.11.0.2 iburst prefer
driftfile /var/lib/chrony/drift

systemctl enable --now chronyd


HQSRV 
2G!
/etc/resolv.conf search hq.work
ipa-server-install --mkhomedir
reboot
kinit admin

BR-SRV
1+1
lsblk
fdisk /dev/...
n
p
1


t
fd
w
mdadm --create /dev/md0 --level=5 --raid-devices=2 /dev/sd[b-c]
mdadm -D /dev/md0
mkfs.ext4 /dev/md0
mkdir /mnt/raid5
mount /dev/md0 /mnt/raid5
echo "/dev/md0 /mnt/raid5 ext4 defaults 0 0" >> /etc/fstab

NyashMan/DEMO2024/

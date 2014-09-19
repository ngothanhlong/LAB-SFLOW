
1. Mục tiêu bài lab

  - Moitor lưu lượng mạng gửi tới và đi của VM1, VM2 sử dụng công nghệ sFlow
  - Tìm hiểu được cách sử dụng công cụ sFlow để monitor Open vSwitch
  - Hiểu được các kết quả trả về của công cụ sFlow
2.Mô hình LAB :
  
   <img src="http://i.imgur.com/s889R4C.png">
 
 *. Cấu hình mạng cho HOST1:
 <img src="http://i.imgur.com/aC8MzYs.png">
3. Cài đặt: .
Cài đặt các gói sau:
- Cai goi KVM :
```
sudo apt-get install -y kvm libvirt-bin pm-utils
```
- Cai goi openvswitch :
``` 
 apt-get install -y openvswitch-switch openvswitch-datapath-dkms
```
Tạo brigde br0:
```
 sudo ovs-vsctl add-br br0
```
Thêm port eth0 vào br0 :
```
 ovs-vsctl add-port br0 eth0
```
Sau khi thực hiện câu lệnh trên thì card eth0 sẽ mất kết nối, ta cần xóa IP của card eth0 và lấy IP đó đặt cho card br0
```
ifconfig eth0 0.0.0.0
ifconfig br0 172.16.69.25/24
```

Tạo hai máy ảo:
```
wget http://cdn.download.cirros-cloud.net/0.3.2/cirros-0.3.2-x86_64-disk.img
mv cirros-0.3.0-x86_64-disk.img cirros1.img
cp cirros1.img cirros2.img
kvm -m 512 -net nic,macaddr=12:42:52:CC:CC:12 -net tap cirros1.img  –nographic
kvm -m 512 -net nic,macaddr=12:42:52:CC:CC:13 -net tap cirros2.img  –nographic
```
Sau khi tạo mỗi VM thì sẽ bị mất phiên ssh vào máy chủ, thay vào đó sẽ là phiên của mỗi máy ảo, ta cần mở một kết nối khác tới máy chủ để cấu hình.

Ta tạo một file mới có tên là sflow.sh với nội dung như sau:

```
#Set parameters switch to connect to COLLECTOR
COLLECTOR_IP=10.0.0.1
COLLECTOR_PORT=6343
AGENT_IP=eth1
HEADER_BYTES=128
SAMPLING_N=64
POLLING_SECS=1
#Virtual Switch is monitored
BRIDGE=br0
```
#Configure Sflow in virtual Switch:
```
ovs-vsctl -- --id=@sflow create sflow agent=${AGENT_IP} target=\"${COLLECTOR_IP}:${COLLECTOR_PORT}\" header=${HEADER_BYTES} sampling=${SAMPLING_N} polling=${POLLING_SECS} -- set  bridge ${BRIDGE} sflow=@sflow
```
Với COLLECTOR_IP là IP của monitoring host, eth1 là card mạng nối với monitoring host.

```
chmod +x sflow.sh
./sflow.sh
```
- Show tất cả Sflow đã đặt trên Open vSwitch:
```
ovs-vsctl list sflow
```
- Xóa sflow trên Open vSwitch
```
ovs-vsctl remove bridge br0 sflow $SFLOWUUID
```
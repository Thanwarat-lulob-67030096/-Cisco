1️⃣ External Router (จำลอง Internet / ISP)
enable
configure terminal

! Interface เชื่อม R1
interface g0/0/0
ip address 200.200.200.1 255.255.255.252
no shutdown
exit

! Interface เชื่อม WebServer
interface g0/0/1
ip address 200.200.100.1 255.255.255.252
no shutdown
exit

! Routing กลับไป LAN
ip route 192.96.0.0 255.255.0.0 200.200.200.2

end
write memory


2️⃣ WebServer / DNS Server จำลอง
IP: 200.200.100.2
Subnet: 255.255.255.252
Gateway: 200.200.100.1


3️⃣ R1 (Router หน่วยงาน) + NAT + ACL
enable
configure terminal

! Interface เชื่อม LAN (CoreSW)
interface g0/0/1
ip address 192.96.3.1 255.255.255.252
no shutdown
exit

! Interface เชื่อม External Router
interface g0/0/0
ip address 200.200.200.2 255.255.255.252
no shutdown
exit

! ACL NAT - อนุญาต VLAN20 ออก Internet, บล็อก VLAN10
ip access-list standard NAT_LAN
 permit 192.96.2.0 0.0.0.255
exit

! NAT Overload (PAT) ใช้ Public IP เดียวออก Internet
ip nat inside source list NAT_LAN interface g0/0/0 overload

! กำหนด Inside / Outside
interface g0/0/1
ip nat inside
exit

interface g0/0/0
ip nat outside
exit

! Static route default ไป External Router
ip route 0.0.0.0 0.0.0.0 200.200.200.1

! Route กลับไป LAN
ip route 192.96.1.0 255.255.255.0 192.96.3.2
ip route 192.96.2.0 255.255.255.0 192.96.3.2

end
write memory


4️⃣ สิ่งที่ต้องตรวจสอบหลัง config
- PC VLAN20 สามารถ ping ออกไป WebServer ได้
- PC VLAN10 ต้องถูกบล็อก (ping ไม่ผ่าน)
- ตรวจสอบ NAT ด้วย `show ip nat translations`
- ตรวจสอบ routing ด้วย `show ip route`
- ตรวจสอบ interface ด้วย `show ip interface brief`

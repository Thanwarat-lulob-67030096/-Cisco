# ทดสอบด้วยการ ping จาก PC ทุกตัวไปยัง WebServer 

---

### 🔹 SW1 (เหมือนกันกับ SW2–SW4 แค่เปลี่ยนพอร์ตต่อ PC ให้ตรง)

```bash
enable
configure terminal

! VLAN
vlan 10
exit
vlan 20
exit

! trunk uplink ไป CoreSW
interface fa0/1
switchport mode trunk
exit

! Access ports
interface fa0/2
switchport mode access
switchport access vlan 10
exit

interface fa0/3
switchport mode access
switchport access vlan 20
exit

end
write memory
```

---

### 🔹 Core Switch (3560)

```bash
enable
configure terminal

! VLAN + SVI
vlan 10
exit
vlan 20
exit

interface vlan 10
ip address 192.96.1.1 255.255.255.0
no shutdown
exit

interface vlan 20
ip address 192.96.2.1 255.255.255.0
no shutdown
exit

! เปิด IP Routing
ip routing

! Trunk ไป Access Switches
interface fa0/1
switchport mode trunk
exit

! เชื่อม R1
interface gi0/1
no switchport
ip address 192.96.3.2 255.255.255.252
no shutdown
exit

! Default Route ไป R1
ip route 0.0.0.0 0.0.0.0 192.96.3.1

end
write memory
```

---

### 🔹 Router R1 (ISR4331)

```bash
enable
configure terminal

! เชื่อม CoreSW
interface g0/0/1
ip address 192.96.3.1 255.255.255.252
no shutdown
exit

! เชื่อม ISP
interface g0/0/0
ip address 200.200.200.2 255.255.255.252
no shutdown
exit

! Default Route ออก ISP
ip route 0.0.0.0 0.0.0.0 200.200.200.1

! Static Route กลับไป LAN
ip route 192.96.1.0 255.255.255.0 192.96.3.2
ip route 192.96.2.0 255.255.255.0 192.96.3.2

end
write memory
```

---

### 🔹 ISP Router

```bash
enable
configure terminal

! เชื่อม R1
interface g0/0/0
ip address 200.200.200.1 255.255.255.252
no shutdown
exit

! เชื่อม WebServer
interface g0/0/1
ip address 200.200.100.1 255.255.255.252
no shutdown
exit

! Default Route กลับไป R1
ip route 0.0.0.0 0.0.0.0 200.200.200.2

end
write memory
```

---

### 🔹 Web Server (ตั้งค่าใน Desktop → IP Config)

```
IP Address: 200.200.100.2
Subnet Mask: 255.255.255.252
Gateway: 200.200.100.1
```

---

### 🔹 PC Config (ตั้งค่าใน Desktop → IP Config)

| PC  | IP         | Subnet Mask   | Gateway    |
| --- | ---------- | ------------- | ---------- |
| PC1 | 192.96.1.2 | 255.255.255.0 | 192.96.1.1 |
| PC2 | 192.96.2.2 | 255.255.255.0 | 192.96.2.1 |
| PC3 | 192.96.1.3 | 255.255.255.0 | 192.96.1.1 |
| PC4 | 192.96.2.3 | 255.255.255.0 | 192.96.2.1 |
| PC5 | 192.96.1.4 | 255.255.255.0 | 192.96.1.1 |
| PC6 | 192.96.2.4 | 255.255.255.0 | 192.96.2.1 |
| PC7 | 192.96.1.5 | 255.255.255.0 | 192.96.1.1 |
| PC8 | 192.96.2.5 | 255.255.255.0 | 192.96.2.1 |

---

### ✅ Test Command

```bash
ping 192.96.1.1
ping 192.96.2.1
ping 200.200.100.2
```

---

คุณอยากให้ผมรวมทุก Config (SW1–SW4, CoreSW, R1, ISP) เป็น **ไฟล์เดียว .txt หรือ .cfg** ที่กด Copy แล้วไป Paste ได้ยาว ๆ เลยไหมครับ?

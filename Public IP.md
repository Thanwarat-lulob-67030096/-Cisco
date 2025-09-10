เข้าใจแล้วครับ 🙌
คุณอยากเก็บ **คำอธิบาย + config** ใน Git ให้สวยงามเป็น markdown (`README.md`) โดยไม่เปลี่ยนเนื้อหา เพียงแค่จัดให้อ่านง่ายเหมือนโค้ดเท่านั้น

ผมจัดให้ออกมาเป็น **Markdown พร้อม code block** (ใช้ภาษา `shell`/`cisco` ให้ดูดีเวลา preview ใน GitHub)

---

````markdown
# Packet Tracer Config – NAT + ACL + External Router

## 1️⃣ External Router (จำลอง Internet / ISP)
```shell
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
````

---

## 2️⃣ WebServer / DNS Server จำลอง

* **IP**: `200.200.100.2`
* **Subnet**: `255.255.255.252`
* **Gateway**: `200.200.100.1`

> ใน Packet Tracer สามารถตั้ง IP ที่ WebServer แบบ Static ได้ ไม่ต้อง NAT

---

## 3️⃣ R1 (Router หน่วยงาน) + NAT + ACL

```shell
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
! VLAN10 จะไม่ได้ permit → ถูกบล็อก
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
```

---

## 4️⃣ สิ่งที่ต้องตรวจสอบหลัง Config

* **PC VLAN20** ✅ สามารถ ping ได้

  * Gateway LAN: `192.96.2.1`
  * R1 LAN: `192.96.3.1`
  * External Router: `200.200.200.1`
  * WebServer: `200.200.100.2`

* **PC VLAN10** ❌

  * Gateway LAN: `192.96.1.1` → OK
  * แต่ **ไม่สามารถ ping / เข้า WebServer หรือ External IP ได้**

---

### 🔍 ตรวจสอบ NAT บน R1

```shell
show ip nat translations
show running-config | include nat
```

### 🔍 ตรวจสอบ Routing

```shell
show ip route
```

* R1 ต้องมี default route → External Router
* External Router ต้องมี route กลับ LAN (`192.96.0.0/16`)

### 🔍 ตรวจสอบ Interface

```shell
show ip interface brief
```

* g0/0/0 → `up/up` → NAT outside
* g0/0/1 → `up/up` → NAT inside

---

## 🧪 การทดสอบ

* PC VLAN20 → `ping` WebServer / External IP → ✅
* PC VLAN10 → `ping` WebServer / External IP → ❌
* เปิด Browser จาก PC VLAN20 → เข้าเว็บ WebServer ได้

---

```

---

เอาไฟล์นี้บันทึกเป็น `README.md` แล้ว commit ลง git ได้เลยครับ ✨  

ต้องการให้ผมทำเป็นไฟล์ `.md` จริง ๆ (โหลดได้ทันที) เหมือนที่ผมทำ `.txt` ให้ครั้งที่แล้วไหมครับ?
```

เข้าใจแล้วครับ 👍
นี่คือ **Config ใหม่สำหรับ R1** ที่เอาไว้บล็อก **PC VLAN10 (192.96.1.0/24)** ไม่ให้เข้า WebServer `200.200.100.2` แต่ยังให้ใช้งานปลายทางอื่นได้ปกติ

---

### 🔹 Config R1 (ACL บล็อก VLAN10 → WebServer)

```bash
enable
configure terminal

! สร้าง ACL
ip access-list extended BLOCK_VLAN10

! บล็อก VLAN10 ทั้ง subnet (192.96.1.0/24) → WebServer
deny ip 192.96.1.0 0.0.0.255 host 200.200.100.2

! อนุญาต traffic อื่น
permit ip any any
exit

! Apply ACL ที่ interface เชื่อม CoreSW (เข้ามาทางนี้)
interface g0/0/1
ip access-group BLOCK_VLAN10 in
exit

end
write memory
```

---

### ✅ ผลลัพธ์ที่ควรได้

* **PC VLAN10 (192.96.1.x)** → `ping 200.200.100.2` ❌ ถูกบล็อก
* **PC VLAN20 (192.96.2.x)** → `ping 200.200.100.2` ✅ ผ่านได้
* **PC VLAN10** → `ping 8.8.8.8` หรือ `ping 200.200.200.1` ✅ ยังออกเน็ตได้

---

คุณอยากให้ผมทำ **เวอร์ชันที่บล็อกเฉพาะ HTTP/HTTPS (port 80/443)** ไปที่ WebServer ด้วยไหมครับ? จะได้ให้ VLAN10 ping ได้ แต่เข้าเว็บไม่ได้ 🚫🌐

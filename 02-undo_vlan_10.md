# ถ้าหน่วยงานไม่ต้องการให้เครื่องคอมพิวเตอร์ภายในเครือข่าย VLAN10 สื่อสารกับ WebServer ได้ ให้ นักศึกษาตั้งค่าเครือข่ายตามเงื่อนไขดังกล่าว

### เข้าเว็ป
```bash
http://200.200.100.2
```

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


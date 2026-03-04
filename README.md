# update-server-ip

> Bulk อัพเดท **Server IP** ใน LiteSpeed Cache plugin ทุกเว็บใน cPanel/WHM server  
> รันครั้งเดียว อัพเดทอัตโนมัติทุกเว็บพร้อมกัน

---

## วิธีรัน

```bash
bash <(curl -s https://raw.githubusercontent.com/ufavision/litespeed-update-server-ip/main/update-server-ip)
```

---

## ทำอะไร

อัพเดทค่า **Server IP** ที่:
```
LiteSpeed Cache › General › Server IP
```

### ขั้นตอนการทำงาน

| # | สิ่งที่ทำ |
|---|----------|
| 1 | ดึง Server IP ปัจจุบันจาก `ifconfig.me` |
| 2 | ค้นหา WordPress ทุกเว็บใน `/home/*/public_html/*/` |
| 3 | ตรวจ `litespeed-cache` plugin installed |
| 4 | ตรวจ `litespeed-cache` plugin active |
| 5 | อัพเดท `litespeed.conf.server_ip` = Server IP ใหม่ |
| 6 | Verify ค่าที่บันทึกถูกต้อง |
| 7 | log ผล success / failed / skipped |

---

## สรุปผลรวม (แสดงหลังรันเสร็จ)

```
======================================
 สรุปผล
 ✅ Success : 800 เว็บ
 ⏭  Skipped : 100 เว็บ
 ❌ Failed  :  10 เว็บ
 รวม        : 910 เว็บ
 Log อยู่ที่ : /var/log/lscwp-update-ip.log
======================================
```

### อธิบายแต่ละสถานะ

| สถานะ | ความหมาย |
|-------|---------|
| ✅ **Success** | อัพเดท Server IP สำเร็จ และ verify ค่าตรงแล้ว |
| ⏭️ **Skipped** | plugin ไม่ได้ install หรือ plugin ไม่ active |
| ❌ **Failed** | อัพเดทไม่สำเร็จ หรือ verify แล้วค่าไม่ตรง |

---

## Log File

| ไฟล์ | เนื้อหา |
|------|---------|
| `/var/log/lscwp-update-ip.log` | log รวมทุก event |

---

## คำสั่งดู Log

### ดู real-time ขณะรัน
```bash
tail -f /var/log/lscwp-update-ip.log
```

### ดู log ทั้งหมด
```bash
cat /var/log/lscwp-update-ip.log
```

### ดูเฉพาะ ✅ Success
```bash
grep "SUCCESS" /var/log/lscwp-update-ip.log
```

### ดูเฉพาะ ❌ Failed
```bash
grep "FAILED" /var/log/lscwp-update-ip.log
```

### ดูเฉพาะ ⏭️ Skipped
```bash
grep "SKIPPED" /var/log/lscwp-update-ip.log
```

### นับจำนวนแต่ละประเภท
```bash
echo "✅ Success : $(grep -c "SUCCESS" /var/log/lscwp-update-ip.log)"
echo "❌ Failed  : $(grep -c "FAILED"  /var/log/lscwp-update-ip.log)"
echo "⏭  Skipped : $(grep -c "SKIPPED" /var/log/lscwp-update-ip.log)"
```

### ดูเฉพาะวันนี้
```bash
grep "$(date '+%Y-%m-%d')" /var/log/lscwp-update-ip.log
```

### ค้นหาเว็บที่ต้องการ
```bash
grep "domain.com" /var/log/lscwp-update-ip.log
```

### ล้าง log ก่อนรันใหม่
```bash
> /var/log/lscwp-update-ip.log
```

---

## Performance

- **Parallel jobs** — คำนวณอัตโนมัติจาก CPU cores + RAM ที่เหลืออยู่ (สูงสุด 20 jobs)
- **Auto MAX_JOBS** — `RAM ÷ 200MB` vs `CPU cores` เลือกค่าที่น้อยกว่า

---

## License

MIT

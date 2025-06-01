# 🍓 คู่มือการตั้งค่า Raspberry Pi Network & Bluetooth

คู่มือครอบคลุมสำหรับการตั้งค่า WiFi และ Bluetooth บน Raspberry Pi พร้อมการ์ดไร้สาย Intel AX210

## 📋 สารบัญ

- [ข้อกำหนดเบื้องต้น](#-ข้อกำหนดเบื้องต้น)
- [การติดตั้งแพ็คเกจเริ่มต้น](#-การติดตั้งแพ็คเกจเริ่มต้น)
- [การตั้งค่า PCIe](#-การตั้งค่า-pcie)
- [การตั้งค่า WiFi](#-การตั้งค่า-wifi)
- [การตั้งค่า Bluetooth](#-การตั้งค่า-bluetooth)
- [การตรวจสอบผลลัพธ์](#-การตรวจสอบผลลัพธ์)
- [การแก้ปัญหา](#-การแก้ปัญหา)

## 🛠 ข้อกำหนดเบื้องต้น

- Raspberry Pi ที่มีช่อง PCIe
- การ์ดไร้สาย Intel AX210 (หรือที่เข้ากันได้)
- Raspberry Pi OS ที่ติดตั้งใหม่
- การเข้าถึง Terminal (SSH หรือตรง)

## 📦 การติดตั้งแพ็คเกจเริ่มต้น

ติดตั้งแพ็คเกจที่จำเป็นสำหรับเครือข่ายและบลูทูธ:

```bash
sudo apt update
sudo apt install net-tools
sudo apt install network-manager
sudo apt install bluez
```

## ⚙️ การตั้งค่า PCIe

กำหนดค่า PCIe เพื่อเปิดใช้งานการ์ดไร้สาย:

1. แก้ไขไฟล์กำหนดค่าการบูต:
```bash
sudo nano /boot/firmware/config.txt
```

2. เพิ่มบรรทัดเหล่านี้หลังจากส่วน `[all]`:
```
dtparam=pciex1
dtparam=pciex1_gen=3
```

3. รีบูตระบบ:
```bash
sudo reboot
```

## 📶 การตั้งค่า WiFi

### ขั้นตอนที่ 1: ตรวจสอบสถานะอุปกรณ์
ตรวจสอบว่าไดรเวอร์ไร้สายถูกตรวจพบหรือไม่:
```bash
nmcli d status
```

### ขั้นตอนที่ 2: เปิดใช้งานการจัดการอุปกรณ์
ตั้งค่าอุปกรณ์ไร้สายให้ถูกจัดการโดย NetworkManager:
```bash
sudo nmcli d set wlp1s0 managed yes
```

### ขั้นตอนที่ 3: สแกนหาเครือข่าย
สแกนหาเครือข่าย WiFi ที่มีอยู่:
```bash
sudo nmcli d wifi rescan ifname wlp1s0
sudo nmcli d wifi list ifname wlp1s0
```

### ขั้นตอนที่ 4: เชื่อมต่อ WiFi
เชื่อมต่อกับเครือข่ายที่ต้องการ:
```bash
sudo nmcli d wifi connect "ชื่อ_wifi" password "รหัสผ่าน_wifi" ifname wlp1s0
```
> แทนที่ `ชื่อ_wifi` และ `รหัสผ่าน_wifi` ด้วยข้อมูลจริงของเครือข่าย

### ขั้นตอนที่ 5: ตั้งค่าลำดับความสำคัญเครือข่าย
กำหนดค่า route metrics สำหรับการเชื่อมต่อ AX210:
```bash
sudo nmcli c modify "ชื่อ_wifi" ipv4.route-metric 50
```

### ขั้นตอนที่ 6: รีสตาร์ทการเชื่อมต่อ
ใช้การตั้งค่าใหม่:
```bash
sudo nmcli c down "ชื่อ_wifi"
sudo nmcli c up "ชื่อ_wifi"
```

## 🔵 การตั้งค่า Bluetooth

### ขั้นตอนที่ 1: เข้าสู่การควบคุม Bluetooth
เปิดการใช้งานอินเทอร์เฟซควบคุม Bluetooth:
```bash
bluetoothctl
```

### ขั้นตอนที่ 2: สแกนหาอุปกรณ์
เริ่มสแกนหาอุปกรณ์ Bluetooth ใกล้เคียง:
```bash
scan on
```
รอให้อุปกรณ์ปรากฏในผลการสแกน

### ขั้นตอนที่ 3: แสดงรายการอุปกรณ์ที่มี
ดูอุปกรณ์ที่ค้นพบ:
```bash
devices
```

### ขั้นตอนที่ 4: จับคู่และเชื่อมต่อ
สำหรับแต่ละอุปกรณ์ที่ต้องการเชื่อมต่อ ให้รันคำสั่งเหล่านี้:
```bash
pair [MAC_ADDRESS]
connect [MAC_ADDRESS]
trust [MAC_ADDRESS]
```
> แทนที่ `[MAC_ADDRESS]` ด้วย MAC address จริงของอุปกรณ์

### ขั้นตอนที่ 5: ตรวจสอบคอนโทรลเลอร์
ตรวจสอบข้อมูลคอนโทรลเลอร์ Bluetooth:
```bash
show
```

## ✅ การตรวจสอบผลลัพธ์

### ผลลัพธ์ที่คาดหวังของคอนโทรลเลอร์ Bluetooth
หากการตั้งค่าสำเร็จ คุณควรเห็นผลลัพธ์คล้ายกับนี้:

```
Controller MAC: C8:58:B3:7A:3E:EB = AX210 (Intel)
Manufacturer: 0x0002 (2) = Intel Corp.
Version: 0x0c (12) = Bluetooth 5.3
Modalias: usb:v1D6Bp0246d0548 = USB device
```

### ขั้นตอนที่ 6: ออก
```bash
exit
```

### การตรวจสอบสถานะเครือข่าย
ตรวจสอบการเชื่อมต่อ WiFi:
```bash
nmcli connection show
ping google.com
```

## 🔧 การแก้ปัญหา

### ปัญหาที่พบบ่อย

**ไม่พบ WiFi:**
- ตรวจสอบว่าการกำหนดค่า PCIe ถูกต้อง
- ตรวจสอบว่าการ์ดไร้สายติดตั้งอย่างถูกต้อง
- ตรวจสอบว่าโหลดเฟิร์มแวร์แล้ว: `dmesg | grep iwlwifi`

**Bluetooth ไม่ทำงาน:**
- รีสตาร์ทบริการ Bluetooth: `sudo systemctl restart bluetooth`
- ตรวจสอบว่าอุปกรณ์ถูกบล็อก: `rfkill list`
- ปลดบล็อกหากจำเป็น: `sudo rfkill unblock bluetooth`

**การเชื่อมต่อขาดหาย:**
- ตรวจสอบความแรงของสัญญาณ: `iwconfig`
- ตรวจสอบ log ระบบ: `journalctl -u NetworkManager`

### คำสั่งที่มีประโยชน์

```bash
# ตรวจสอบอินเทอร์เฟซไร้สาย
ip link show

# ตรวจสอบการเชื่อมต่อ
watch -n 1 'nmcli d status'

# รีเซ็ต NetworkManager
sudo systemctl restart NetworkManager

# ตรวจสอบสถานะ Bluetooth
systemctl status bluetooth
```

## 📝 หมายเหตุ

- Intel AX210 รองรับ WiFi 6E และ Bluetooth 5.3
- Route metric ของ 50 ให้ความสำคัญกับการเชื่อมต่อ AX210
- ควรรีบูตเสมอหลังจากเปลี่ยนแปลงการกำหนดค่า PCIe
- ให้อัพเดทไดรเวอร์อุปกรณ์เพื่อประสิทธิภาพที่ดีที่สุด

---

*อัพเดทล่าสุด: มิถุนายน 2025*

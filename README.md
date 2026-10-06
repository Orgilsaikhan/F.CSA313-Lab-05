# Lab 05 - B222270044 - Г. Оргилсайхан

**Сэдэв:** API систем тест — Postman ба Newman (F.CSA313, 2026)

```
$ node -v
v22.22.1

$ newman -v
6.2.2
```

Орчин: Ubuntu 26.04.1 LTS · Node.js v22.22.1 · Newman 6.2.2

## Товч үр дүн

| Үзүүлэлт | Үр дүн |
|---|---|
| Спецификаци / тест | **11** (S1–S11), collection-д 11 бие даасан folder |
| `results/newman-pass.txt` | requests 26, **assertions 31 executed, 0 failed**, `exit=0` |
| `results/newman-fail.txt` (S1: 201 → 200) | assertions 31 executed, **1 failed**, `exit=1` |
| `results/newman-down.txt` (сервер унтарсан) | `ECONNREFUSED`, requests 26 / 26 failed, `exit=1` |
| `results/newman-public.txt` (Даалгавар 5) | assertions 3 executed, 0 failed, `exit=0` |
| `server.js`-ээс олдсон согог | **3** — давхар бүртгэл, `"toString"` ID буруу алдаа, `"__proto__"` серверийг унагадаг (`results/defects.txt`) |

## Явц

- [x] Алхам 1 — Орчин бэлдэх, `server.js` ажиллуулж curl-ээр шалгах
- [x] Даалгавар 1 — Анхны хүсэлт ба 2 oracle
- [x] Даалгавар 2 — Сонголт, төлөөлөх утга, спецификацийн хүснэгт
- [x] Даалгавар 3 — Collection бүтээх (8+ бие даасан тест)
- [x] Даалгавар 4 — Newman: PASS / FAIL / DOWN
- [x] Даалгавар 5 — Нийтийн API (jsonplaceholder)
- [x] Дүгнэлт

## Репогийн бүтэц

```
server.js                     # тестлэгдэх API (багшийн өгсөн, өөрчлөөгүй)
lab05-collection.json         # 11 тест, 31 assertion — Postman Collection v2.1
lab05-collection-fail.json    # FAIL хувилбар — S1 статусын oracle 201 → 200
lab05-public-collection.json  # Даалгавар 5 — jsonplaceholder GET /users, 3 assertion
results/newman-pass.txt       # Даалгавар 4 — PASS, exit=0
results/newman-fail.txt       # Даалгавар 4 — FAIL, exit=1
results/newman-down.txt       # Даалгавар 4 — DOWN (ECONNREFUSED), exit=1
results/newman-public.txt     # Даалгавар 5 — exit=0
results/independence.txt      # Даалгавар 3 — folder бүрийг дангаар нь + 3 удаа дараалан ажиллуулсан нь
results/defects.txt           # server.js-ийн 3 согогийг curl-ээр баримтжуулсан нь
```

Өөрөө ажиллуулж шалгах бол:

```bash
sudo npm install -g newman
node server.js                                                                # 1-р терминал
newman run lab05-collection.json 2>&1 | tee results/newman-pass.txt           # 2-р терминал
echo "exit=${PIPESTATUS[0]}"
```

---

## Алхам 1: Орчин

```bash
sudo npm install -g newman
node server.js   # -> Бүртгэлийн API: http://localhost:3000
```

Зааврын 3 curl командаар API ажиллаж буйг шалгасан:

```
$ curl -X PUT localhost:3000/students/B231234567 -H 'Content-Type: application/json' -d '{"status":"active","coursesTaken":["CS201"]}'
{"result":"OK"}
$ curl -X PUT localhost:3000/courses/CS313 -H 'Content-Type: application/json' -d '{"prerequisites":["CS201"]}'
{"result":"OK"}
$ curl -X POST localhost:3000/registrations -H 'Content-Type: application/json' -d '{"studentID":"B231234567","courseID":"CS313"}'
{"result":"OK","registrationID":1}
```

## Даалгавар 1: Анхны хүсэлт ба oracle

`lab05-collection.json` (Postman Collection v2.1 — Postman-д File → Import-оор нээгдэнэ) үүсгэж, `baseUrl = http://localhost:3000` collection variable нэмсэн. Бүх URL `{{baseUrl}}/...` хэлбэртэй тул серверийн хаяг солигдвол нэг газар засна.

Эхний folder 3 хүсэлттэй:

| # | Хүсэлт | Бие |
|---|---|---|
| 1 | `PUT {{baseUrl}}/students/B231234567` | `{"status":"active","coursesTaken":["CS201"]}` |
| 2 | `PUT {{baseUrl}}/courses/CS313` | `{"prerequisites":["CS201"]}` |
| 3 | `POST {{baseUrl}}/registrations` | `{"studentID":"B231234567","courseID":"CS313"}` |

POST-ийн Tests таб дахь 2 oracle — статус ба утга:

```js
pm.test("Статус 201 Created", function () {
    pm.response.to.have.status(201);
});

pm.test("result = OK", function () {
    pm.expect(pm.response.json().result).to.eql("OK");
});
```

Энэ folder Даалгавар 3-т S1 болж, `registrationID`-ийн гурав дахь oracle нэмэгдсэн. `results/newman-pass.txt` дахь S1:

```
↳ POST /registrations
  POST http://localhost:3000/registrations [201 Created, 202B, 2ms]
  ✓  Статус 201 Created
  ✓  result = OK
  ✓  registrationID нь тоо
```

## Даалгавар 2: Тест дизайн (Лекц 5-ын 5 алхам)

### Алхам 1 — Бие даан тестлэгдэх функц

`POST /registrations` — оюутныг хичээлд бүртгэх. Оролт нь хүсэлтийн биеийн 2 параметр (`studentID`, `courseID`) **ба** серверийн төлөв: оюутны `status`, `coursesTaken`, хичээлийн `prerequisites`. Төлөвийг тест бүр өөрөө `PUT /students/:id`, `PUT /courses/:id`-ээр тавина. Гаралт нь HTTP статус ба `result` (урьдач нөхцөл дутуу бол `missing` массив).

### Алхам 2–3 — Сонголтууд ба төлөөлөх утгууд

| Сонголт | Анги | Төлөөлөх утга (setup) | Тэмдэг |
|---|---|---|---|
| **A.** `studentID`-гийн хүчинтэй байдал | идэвхтэй | `B231234567`, `"status":"active"` | |
| | идэвхгүй | `B231111111`, `"status":"inactive"` | [алдаа] |
| | байхгүй | `B239999999` — ямар ч тест PUT хийдэггүй | [алдаа] |
| | талбар дутуу | бие: `{"courseID":"CS313"}` | [хязгаар] |
| **B.** Оюутны үзсэн хичээлүүд (`coursesTaken`) | урьдач хангана | `["CS201"]` (`CS313`-ийн хувьд) | |
| | урьдач хангахгүй | `["MATH101"]` | [алдаа] |
| | хоосон | `[]` | [хязгаар] |
| **C.** `courseID`-гийн хүчинтэй байдал | байгаа | `CS313` | |
| | байхгүй | `CS999` — ямар ч тест PUT хийдэггүй | [алдаа] |
| **D.** Хичээлийн урьдач нөхцөл (`prerequisites`) | бүгд үзсэн | `CS313`: `["CS201"]`, үзсэн `["CS201"]` | |
| | огт үзээгүй | `CS401`: `["CS201","CS313"]`, үзсэн `["MATH101"]` | [алдаа] |
| | зарим нь | `CS401`: `["CS201","CS313"]`, үзсэн `["CS201"]` | [алдаа] |
| | урьдач нөхцөлгүй | `CS101`: `[]` | [хязгаар] |
| **E.** Хүсэлтийн формат | буруу JSON | бие: `{"studentID":"B231234567",` | [хязгаар] |

- **[алдаа]** — алдааны анги. Бусад сонголтын бүх утгатай хослуулах шаардлагагүй: нэг удаа дангаар нь, мөн давхар алдааны хослолд шалгана.
- **[хязгаар]** — хязгаарын тохиолдол, тусад нь нэг удаа шалгана.

### Боломжгүй хослолууд

Хязгаарын утгуудгүйгээр нийт хослол: A(3) × B(2) × C(2) × D(3) = **36**. Гэхдээ сонголтууд хоорондоо хамааралтай:

1. **A = байхгүй → B утгагүй.** Бичлэггүй оюутанд "үзсэн хичээл" гэж байхгүй.
2. **C = байхгүй → D утгагүй.** Бичлэггүй хичээлд урьдач нөхцөл байхгүй.
3. **B ба D нь нэг баримтыг хоёр талаас нь хардаг.** B = "хангана" ⇔ D = "бүгд үзсэн". "Урьдач хангана, гэхдээ огт үзээгүй" гэдэг нь өөртэйгөө зөрчилддөг тул (B, D)-ийн 6 хосоос зөвхөн 3 нь боломжтой.

Эдгээрийг хасахад утга бүхий **10** хослол үлдэнэ: A ∈ {идэвхтэй, идэвхгүй} × C = байгаа × 3 (B, D) хос = 6; A ∈ {идэвхтэй, идэвхгүй} × C = байхгүй = 2; A = байхгүй × C ∈ {байгаа, байхгүй} = 2.

### Давхар алдаа — аль нь давуу вэ?

Зааварт бичээгүй тул curl-ээр туршиж тогтоосон: байхгүй оюутан + байхгүй хичээл → `ERROR_NO_STUDENT`, идэвхгүй оюутан + байхгүй хичээл → `ERROR_INACTIVE_STUDENT`, идэвхгүй оюутан + урьдач дутуу → `ERROR_INACTIVE_STUDENT`. Өөрөөр хэлбэл сервер эхлээд **оюутныг** (байгаа эсэх → идэвхтэй эсэх), дараа нь **хичээлийг** (байгаа эсэх → урьдач нөхцөл) шалгадаг. S7, S8 энэ дарааллыг тестээр бэхжүүлнэ: шалгалтын дараалал өөрчлөгдвөл API-гийн хэрэглэгч өөр алдааны код харж эхлэх тул тест унах ёстой.

### Алхам 4 — Спецификациуд

| # | Төрөл | Оюутны setup | Хичээлийн setup | POST бие | Статус | `result` |
|---|---|---|---|---|---|---|
| S1 | Happy path | `B231234567` active `["CS201"]` | `CS313` `["CS201"]` | `B231234567` → `CS313` | **201** | `OK` + `registrationID` тоо |
| S2 | Алдаа: A = байхгүй | — | `CS313` `["CS201"]` | `B239999999` → `CS313` | **200** | `ERROR_NO_STUDENT` |
| S3 | Алдаа: A = идэвхгүй | `B231111111` inactive `["CS201"]` | `CS313` `["CS201"]` | `B231111111` → `CS313` | **200** | `ERROR_INACTIVE_STUDENT` |
| S4 | Алдаа: C = байхгүй | `B232222222` active `["CS201"]` | — | `B232222222` → `CS999` | **200** | `ERROR_NO_COURSE` |
| S5 | Алдаа: D = зарим нь | `B233333333` active `["CS201"]` | `CS401` `["CS201","CS313"]` | `B233333333` → `CS401` | **200** | `ERROR_PREREQUISITES`, missing `["CS313"]` |
| S6 | Алдаа: D = огт үзээгүй | `B234444444` active `["MATH101"]` | `CS401` `["CS201","CS313"]` | `B234444444` → `CS401` | **200** | `ERROR_PREREQUISITES`, missing `["CS201","CS313"]` |
| S7 | Давхар: A = байхгүй + C = байхгүй | — | — | `B239999999` → `CS999` | **200** | `ERROR_NO_STUDENT` |
| S8 | Давхар: A = идэвхгүй + D = огт үзээгүй | `B235555555` inactive `[]` | `CS313` `["CS201"]` | `B235555555` → `CS313` | **200** | `ERROR_INACTIVE_STUDENT` |
| S9 | Хязгаар: урьдач нөхцөлгүй хичээл + хоосон `coursesTaken` | `B236666666` active `[]` | `CS101` `[]` | `B236666666` → `CS101` | **201** | `OK` + `registrationID` тоо |
| S10 | Хязгаар: `studentID` талбар дутуу | — | `CS313` `["CS201"]` | `{"courseID":"CS313"}` | **400** | `ERROR_BAD_REQUEST` |
| S11 | Хязгаар: буруу JSON | — | — | `{"studentID":"B231234567",` | **400** | `ERROR_BAD_JSON` |

Лекцийн стратегиар: happy path 1 (S1), алдааны анги бүрд 1 (S2–S5, D ангид "огт үзээгүй" утгаар S6 нэмсэн), давхар алдаа 2 (S7, S8), хязгаар 3 (S9–S11) — **нийт 11**.

**Хамралт:** A–D сонголтын анги бүр дор хаяж нэг спецификацид орсон. Утга бүхий 10 хослолоос 8-ыг нь S1–S8 хамарсан. Хамраагүй 2 нь (идэвхгүй + урьдач "зарим нь", идэвхгүй + байхгүй хичээл) идэвхгүй оюутны алдаа бусдыгаа далдалдаг тул S3, S8-тай ижил `ERROR_INACTIVE_STUDENT` өгнө — [алдаа] ангийг дахин хослуулах шаардлагагүй.

### Алхам 5 — Бодит тест кейс

→ Даалгавар 3: спецификаци бүр collection-д тусдаа folder (тест) болно.

## Даалгавар 3: Collection

`lab05-collection.json` нь **11 folder = 11 тест** (S1–S11). Folder бүр: setup `PUT`-ууд (0–2) → `POST /registrations` → Tests таб дахь oracle-ууд. Даалгавар 1-ийн folder нь S1 болж, `registrationID`-ийн oracle нэмэгдсэн.

```
Lab05 — Хичээлд бүртгүүлэх API          {{baseUrl}} = http://localhost:3000
├── S1 — Идэвхтэй, урьдач нөхцөл хангасан оюутан бүртгүүлэхэд 201 OK
│   ├── setup: PUT /students/B231234567
│   ├── setup: PUT /courses/CS313
│   └── POST /registrations              ✓ Статус 201 Created  ✓ result = OK  ✓ registrationID нь тоо
├── S2 — Байхгүй оюутан бүртгүүлэхэд ERROR_NO_STUDENT
├── S3 — Идэвхгүй оюутан бүртгүүлэхэд ERROR_INACTIVE_STUDENT
├── ...
└── S11 — Буруу JSON илгээхэд 400 ERROR_BAD_JSON
```

### Oracle-ууд

| Oracle | Хаана | Код |
|---|---|---|
| Статус | бүх 11 тест | `pm.response.to.have.status(201)` |
| `result` утга | бүх 11 тест | `pm.expect(pm.response.json().result).to.eql("ERROR_NO_STUDENT")` |
| `registrationID` нь тоо | S1, S9 (амжилттай) | `pm.expect(json.registrationID).to.be.a("number")` |
| Бүртгэл үүсээгүй | S2, S3, S4, S7, S8 | `pm.expect(json).to.not.have.property("registrationID")` |
| `missing` массив | S5, S6 | `pm.expect(json.missing).to.eql(["CS201","CS313"])` |

**Assertion-ы тоо:** 9 тест × 3 + 2 тест (S10, S11) × 2 = **31**.

`registrationID`-ийн **яг утгыг шалгаагүй** — сервер in-memory тул ажиллуулах бүрд 1, 2, 3, … гэж хуримтлагдана. Зөвхөн тоо мөн эсэхийг шалгадаг тул collection-ийг серверийг дахин асаалгүй хэдэн ч удаа ажиллуулж болно.

Алдааны тестүүдэд статус + `result`-ээс гадна **"бүртгэл үүсээгүй"** oracle нэмсэн. Статус ба `result` зөв боловч сервер алдааны үед ч бүртгэл үүсгээд `registrationID` буцаадаг байвал энэ oracle л барина.

### Тест бүр бие даасан

- Тест бүр **өөрийн** setup-ыг өөрөө хийдэг. Оюутны ID тест бүрт өөр (`B231234567`, `B231111111`, …), хичээл (`CS313`, `CS401`) давхцаж болох ч тест бүр POST-оос яг өмнө өөрт хэрэгтэй утгаар нь дахин PUT хийдэг.
- "Байхгүй" утгууд (`B239999999`, `CS999`) — ямар ч тест эдгээрийг PUT хийдэггүй тул үргэлж байхгүй хэвээр.
- Шалгасан нь (`results/independence.txt`): folder бүрийг **дангаар нь, шинээр асаасан (хоосон) сервер дээр** `newman run lab05-collection.json --folder "S5 — ..."` гэж ажиллуулахад 11-үүлээ exit 0. Мөн нэг сервер дээр collection-ийг 3 удаа дараалан ажиллуулахад 3 удаа 31 executed, 0 failed.

```
S1   assertions executed=3 failed=0 exit=0  # S1 — Идэвхтэй, урьдач нөхцөл хангасан оюутан бүртгүүлэхэд 201 OK
...
S10  assertions executed=2 failed=0 exit=0  # S10 — studentID талбар дутуу бол 400 ERROR_BAD_REQUEST
S11  assertions executed=2 failed=0 exit=0  # S11 — Буруу JSON илгээхэд 400 ERROR_BAD_JSON

run 1: assertions executed=31 failed=0 exit=0
run 2: assertions executed=31 failed=0 exit=0
run 3: assertions executed=31 failed=0 exit=0
```

> **Ажиглалт:** 3 удаа ажиллуулсны дараа `GET /registrations` дээр `B231234567 → CS313` **3 удаа** бүртгэгдсэн байсан (`registrationID` 1, 3, 5). Сервер давхар бүртгэлийг шалгадаггүй — Дүгнэлтэд дэлгэрэнгүй.

## Даалгавар 4: Newman

```bash
mkdir -p results
newman run lab05-collection.json 2>&1 | tee results/newman-pass.txt; echo "exit=${PIPESTATUS[0]}" | tee -a results/newman-pass.txt
newman run lab05-collection-fail.json 2>&1 | tee results/newman-fail.txt; echo "exit=${PIPESTATUS[0]}" | tee -a results/newman-fail.txt
# серверийг Ctrl+C-ээр унтраасны дараа:
newman run lab05-collection.json 2>&1 | tee results/newman-down.txt; echo "exit=${PIPESTATUS[0]}" | tee -a results/newman-down.txt
```

Lab 03-аас сурсанчлан `$?` биш `${PIPESTATUS[0]}` ашигласан (`| tee`-ийн дараах `$?` нь tee-ийн 0). Exit code-ийг файлын төгсгөлд `exit=N` мөрөөр хавсаргасан тул нотолгоо `results/` дотроо байна. Newman гаралтыг pipe руу явуулахад өнгөний ANSI код оруулдаггүй тул файлууд цэвэр текст.

| Ажиллуулалт | Collection | requests (executed / failed) | assertions (executed / failed) | Exit code |
|---|---|---|---|---|
| **PASS** — `results/newman-pass.txt` | `lab05-collection.json` | 26 / 0 | **31 / 0** | **0** |
| **FAIL** — `results/newman-fail.txt` | `lab05-collection-fail.json` | 26 / 0 | **31 / 1** | **1** |
| **DOWN** — `results/newman-down.txt` | `lab05-collection.json` | 26 / **26** | 31 / 31 | **1** |

### PASS

11 тест, **31 assertion executed, 0 failed, exit 0**.

### FAIL

`lab05-collection-fail.json` нь үндсэн collection-ийн хуулбар — зөвхөн S1-ийн статусын oracle-ийг зориуд буруу болгосон (үндсэн `lab05-collection.json` зөв хэвээр):

```diff
-pm.test("Статус 201 Created", function () {
-    pm.response.to.have.status(201);
+pm.test("Статус 200 OK", function () {
+    pm.response.to.have.status(200);
```

```
 1.  AssertionError  Статус 200 OK
                     expected response to have status code 200 but got 201
                     inside "S1 — Идэвхтэй, урьдач нөхцөл хангасан оюутан бүртгүүлэхэд 201 OK / POST /registrations"
exit=1
```

**31 executed, 1 failed, exit 1.** CI quality gate-ийн зарчим: Newman ганц assertion унахад л 0 биш exit code буцаадаг тул CI pipeline тэр алхам дээр зогсож, буруу код merge/deploy хийгдэхгүй.

### DOWN

Серверийг Ctrl+C-ээр унтраагаад үндсэн collection-ийг ажиллуулсан. Хүсэлт бүр `connect ECONNREFUSED 127.0.0.1:3000` (файлд 52 мөр) → **requests 26 / 26 failed, exit 1**.

```
↳ POST /registrations
  POST http://localhost:3000/registrations [errored]
     connect ECONNREFUSED 127.0.0.1:3000
  4. Статус 201 Created
  5. result = OK
  6. registrationID нь тоо
```

**Интерфейсийн алдаа ба oracle-ийн алдааны ялгаа:** DOWN нь интерфейсийн алдаа — хүсэлт серверт хүрээгүй (TCP холболт татгалзсан) тул шалгах хариу огт байхгүй, харин FAIL нь oracle-ийн алдаа — сервер хариулсан ч хариу (201) хүлээлттэй (200) зөрсөн.

Хоёулаа exit 1 өгдөг тул exit code-оор ялгах боломжгүй — `requests failed` мөрийг харна: FAIL-д 0 (сервер бүх хүсэлтэд хариулсан), DOWN-д 26. DOWN-ийн "assertions 31 failed" нь жинхэнэ oracle-ийн алдаа биш: мессежүүд нь `expected PostmanResponse{ …(5) } to have property 'code'` (11) ба `"undefined" is not valid JSON` (20) — хариу байхгүй тул oracle үнэлэх юмгүй болсон. Нийт 57 failure = 26 холболтын алдаа + 31 хариугүй assertion.

## Даалгавар 5: Нийтийн API

`lab05-public-collection.json` — **тусдаа** collection (`{{publicUrl}} = https://jsonplaceholder.typicode.com`). Үндсэн collection-д нэмээгүйн шалтгаан: PASS ажиллуулалт интернетээс хамааралгүй байх ба README-ийн 31 assertion-ы тоо зөвхөн өөрийн API-г хэмжинэ.

```js
pm.test("Статус 200 OK", function () {
    pm.response.to.have.status(200);
});

pm.test("Хариу нь массив", function () {
    pm.expect(pm.response.json()).to.be.an("array");
});

pm.test("Эхний хэрэглэгчийн нэр = Leanne Graham", function () {
    pm.expect(pm.response.json()[0].name).to.eql("Leanne Graham");
});
```

```bash
newman run lab05-public-collection.json 2>&1 | tee results/newman-public.txt; echo "exit=${PIPESTATUS[0]}" | tee -a results/newman-public.txt
```

`results/newman-public.txt`: **3 assertion executed, 0 failed, exit 0** (`GET https://jsonplaceholder.typicode.com/users [200 OK, 6.6kB, 407ms]`).

**Өөрийн API-тай харьцуулбал:** GET-only тул setup, тест хоорондын бие даасан байдал гэж санаа зовох зүйлгүй хялбар байсан ч өгөгдлийг би удирддаггүй тул "Leanne Graham" oracle гуравдагч этгээдийн өгөгдөл өөрчлөгдөхөд унаж болох ба идэвхгүй оюутан шиг алдааны төлөвийг өөрөө үүсгэж чадахгүй тул зөвхөн happy path-ийг л шалгаж болсон.

## Олдсон согогууд (`server.js`)

Спецификацийн хүлээгдэх утгуудыг curl-ээр шалгаж байхдаа "байхгүй" ID-нд энгийн бус утга (`toString`, `__proto__`) туршиж үзсэн. Бүгдийг шинээр асаасан сервер дээр давтаж `results/defects.txt`-д хадгалсан.

| # | Оролт | Хүлээгдэх | Бодит | Шалтгаан |
|---|---|---|---|---|
| D1 | Нэг оюутан нэг хичээлд 2 удаа `POST` | 2 дахь нь алдаа (жишээ нь `ERROR_ALREADY_REGISTERED`) — API-гийн тодорхойлолтод заагаагүй | Хоёулаа `201`, `registrationID` 1, 2 | `registrations.push` (мөр 66)-ийн өмнө давхардал шалгадаггүй |
| D2 | `studentID: "toString"` / `"constructor"` (PUT хийгээгүй) | `200 ERROR_NO_STUDENT` | `200 ERROR_INACTIVE_STUDENT` | `students[studentID]` (мөр 51) нь `Object.prototype`-ийн функцийг олж "оюутан байна" гэж үзээд, `status` нь `undefined` тул идэвхгүй гэж дүгнэдэг |
| D3 | `courseID: "__proto__"` | `200 ERROR_NO_COURSE` | **Хариу ирэхгүй, сервер унана** (`TypeError: Cannot read properties of undefined (reading 'filter')`, мөр 59) → дараагийн бүх хүсэлт `ECONNREFUSED` | `courses["__proto__"]` нь `Object.prototype`-г буцаадаг, түүнд `prerequisites` байхгүй |

D3 хамгийн ноцтой: нэвтрэлтгүй ганц хүсэлт бүх хэрэглэгчийн үйлчилгээг зогсооно — Даалгавар 4-ийн DOWN ажиллуулалтыг гаднаас хэн ч үүсгэж чадна гэсэн үг. D2, D3 хоёул `{}` объектыг map болгон ашигласнаас үүдэлтэй — `Map` эсвэл `Object.hasOwn(students, id)` ашиглавал засагдана. PUT endpoint-ууд `[\w-]+` regex-ээр ID-г шалгадаг ч POST-ийн биеийн `studentID`, `courseID`-г огт шалгадаггүй.

`server.js` нь багшийн өгсөн тестлэгдэх систем тул **засаагүй**. Эдгээрийг PASS collection-д тест болгож оруулаагүй: D3-ын тест серверийг унагаж, араас нь ажиллах бүх тестийг эвдэх байсан.

## Дүгнэлт

Лекц 5-ын 5 алхмыг `POST /registrations` дээр хэрэгжүүлж, 4 сонголтоос 11 спецификаци гаргаад collection-д 11 бие даасан тест болгосон бөгөөд `results/newman-pass.txt`-д **31 assertion executed, 0 failed, exit 0** гарсан. Хамгийн их бодол шаардсан нь 2-р алхам буюу сонголтуудыг тодорхойлох байсан: зааврын "оюутны үзсэн хичээлүүд" ба "хичээлийн урьдач нөхцөл" сонголтууд нэг баримтыг хоёр талаас нь хардаг тул тэдгээрийг бие даасан мэт үржүүлбэл утгагүй хослолууд үүсдэг. Гурван боломжгүй хослол таарсан — байхгүй оюутны үзсэн хичээл, байхгүй хичээлийн урьдач нөхцөл, "урьдач нөхцөл хангасан ч огт үзээгүй" — эдгээрийг хасахад 36 хослол 10 болж буурсан. Давхар алдааны үед аль нь буцахыг зааварт бичээгүй тул туршиж тогтоосон: сервер эхлээд оюутныг, дараа нь хичээлийг шалгадаг бөгөөд S7, S8 энэ дарааллыг тестээр бэхжүүлсэн. FAIL ба DOWN хоёулаа exit 1 өгдөг ч FAIL нь oracle-ийн алдаа (сервер хариулсан, утга зөрсөн), DOWN нь интерфейсийн алдаа (хариу огт ирээгүй) бөгөөд тэдгээрийг `requests failed` мөрөөр ялгадаг. Collection-ийг нэг сервер дээр давтан ажиллуулахад нэг оюутан нэг хичээлд хэд хэдэн удаа бүртгэгдсэн нь харагдсан нь давхар бүртгэлийн согог (D1) байв. Хязгаарын утгуудыг гараар туршихад `studentID: "toString"` гэж байхгүй оюутан илгээхэд `ERROR_NO_STUDENT`-ийн оронд `ERROR_INACTIVE_STUDENT` буцдаг (D2), харин `courseID: "__proto__"` гэсэн ганц хүсэлт серверийг бүхэлд нь унагадаг (D3) нь илэрсэн. D3-ын дараа бусад бүх хэрэглэгч DOWN ажиллуулалттай яг адил `ECONNREFUSED` авна — өөрөөр хэлбэл нэг endpoint-ийн oracle-ийн алдаа бүх системийн интерфейсийн алдаа болж хувирсан. Миний спецификацийн хүснэгтэд "байхгүй ID"-г зөвхөн `B239999999` гэсэн энгийн утгаар төлөөлүүлсэн нь "ангийн нэг утга бүх ангийг төлөөлнө" гэсэн эквивалент ангийн таамаглал хэрэгжүүлэлтийн онцлогоос (JavaScript объектын prototype) болж зөрчигдөж болохыг харуулсан. Эндээс сурсан зүйл: тестийн дизайн нь шаардлагаас эхэлдэг ч төлөөлөх утгыг сонгохдоо системийн хэрэгжүүлэлтийн технологи ямар утгыг "онцгой" гэж үздэгийг мөн бодох хэрэгтэй.

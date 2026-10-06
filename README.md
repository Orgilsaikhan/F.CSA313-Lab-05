# Lab 05 - B222270044 - Г. Оргилсайхан

**Сэдэв:** API систем тест — Postman ба Newman (F.CSA313, 2026)

```
$ node -v
v22.22.1

$ newman -v
6.2.2
```

Орчин: Ubuntu 26.04.1 LTS · Node.js v22.22.1 · Newman 6.2.2

## Явц

- [x] Алхам 1 — Орчин бэлдэх, `server.js` ажиллуулж curl-ээр шалгах
- [x] Даалгавар 1 — Анхны хүсэлт ба 2 oracle
- [ ] Даалгавар 2 — Сонголт, төлөөлөх утга, спецификацийн хүснэгт
- [ ] Даалгавар 3 — Collection бүтээх (8+ бие даасан тест)
- [ ] Даалгавар 4 — Newman: PASS / FAIL / DOWN
- [ ] Даалгавар 5 — Нийтийн API (jsonplaceholder)
- [ ] Дүгнэлт

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

```
$ newman run lab05-collection.json
↳ POST /registrations
  POST http://localhost:3000/registrations [201 Created, 202B, 3ms]
  ✓  Статус 201 Created
  ✓  result = OK
...
│              assertions │               2 │               0 │
```


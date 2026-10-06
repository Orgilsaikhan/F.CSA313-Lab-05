# Lab 05 - B222270044 - Г. Оргилсайхан

**Сэдэв:** API систем тест — Postman ба Newman (F.CSA313, 2026)

```
$ node -v
v22.22.1

$ newman -v
NEWMAN_VERSION
```

Орчин: Ubuntu 26.04.1 LTS · Node.js v22.22.1 · Newman NEWMAN_VERSION

## Явц

- [x] Алхам 1 — Орчин бэлдэх, `server.js` ажиллуулж curl-ээр шалгах
- [ ] Даалгавар 1 — Анхны хүсэлт ба 2 oracle
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

# DevOps Session 09 - Bài 3: Dockerfile đóng gói trang web HTML tĩnh

Repo này chứa bài tập số 3 của Session 09 (môn DevOps): viết một `Dockerfile` ngắn gọn để đóng gói
trang `index.html` tĩnh vào image `nginx:alpine`, build thành `my-html-app:v1` và chạy ở cổng 8081.

Báo cáo chi tiết (lệnh, output thật, giải thích, xử lý sự cố): xem
[`homework/session_09/ex3/README.md`](homework/session_09/ex3/README.md).

## Danh sách file

| File | Mô tả |
|------|-------|
| `homework/session_09/ex3/index.html` | Trang web tĩnh, đúng một dòng `<h1>Hello Docker Session 09!</h1>` |
| `homework/session_09/ex3/Dockerfile` | Dockerfile 2 lệnh: `FROM nginx:alpine` và `COPY index.html ...` |
| `homework/session_09/ex3/README.md` | Báo cáo bài tập |
| `homework/session_09/ex3/terminal_log.txt` | Toàn bộ lệnh và output thật khi chạy Docker |
| `homework/session_09/ex3/.dockerignore` | Loại README và log khỏi build context |

# Bài 3 - Session 09: Viết Dockerfile 3 dòng đóng gói trang web HTML tĩnh

## 1. Mục tiêu

- Viết `Dockerfile` siêu ngắn gọn để đóng gói một trang HTML tĩnh thành Docker image.
- Dùng `nginx:alpine` làm image nền, copy `index.html` vào thư mục phục vụ web mặc định của Nginx.
- Build image tên `my-html-app:v1`, chạy container `html-app` ở cổng 8081 (host) -> 80 (container).
- Kiểm tra bằng `curl http://localhost:8081` phải trả về đúng chuỗi `<h1>Hello Docker Session 09!</h1>`.
- Hiểu cấu trúc layer của image và cơ chế layer caching của `docker build`.

Mọi lệnh trong báo cáo đã được chạy thật trên máy; output được chép nguyên văn từ
[`terminal_log.txt`](terminal_log.txt). Riêng log của `nginx` ở mục 5.4 được rút gọn 16 dòng
`start worker process`; docker tự cắt ngắn một số cột dài bằng ký tự dấu ba chấm (đã đổi thành `...` ASCII); pull, history và
một số đoạn log build được lược bớt bằng `...` (có ghi chú tại chỗ).

## 2. Môi trường

| Thành phần | Giá trị |
|------------|---------|
| Hệ điều hành | Windows 11 Home |
| Docker | Docker Desktop 4.89.0, Docker Client/Server 29.7.2, engine Linux qua WSL2 |
| Shell | Git Bash |
| Image nền | `nginx:alpine` (Nginx 1.31.6, Alpine 3.24.2) |
| Cổng | 8081 (host) -> 80 (container) |

```
$ docker info --format '{{.ServerVersion}} {{.OSType}}/{{.Architecture}}'
29.7.2 linux/x86_64
```

Lưu ý Git Bash trên Windows: đặt `export MSYS_NO_PATHCONV=1` trước `docker exec` để đường dẫn `/usr/share/...`
không bị đổi thành `C:/Program Files/Git/...` (xem mục 8). Lần chạy cuối trong log đã đặt biến này.

## 3. Nội dung file

### 3.1. index.html

```
$ cat index.html
<h1>Hello Docker Session 09!</h1>
```

File chỉ có đúng một dòng, kết thúc bằng một ký tự xuống dòng LF (tổng 34 byte). Vì vậy `curl` trả về
đúng chuỗi của đề bài và `diff` với file gốc là giống hệt.

### 3.2. Dockerfile

```
$ cat Dockerfile
# Lệnh 1: chọn image nền Nginx bản Alpine (nhỏ gọn) để làm web server
FROM nginx:alpine
# Lệnh 2: chép trang HTML tĩnh vào thư mục phục vụ web mặc định của Nginx
COPY index.html /usr/share/nginx/html/index.html
```

Tiêu đề đề bài nói "3 dòng", ràng buộc nói "2 câu lệnh": Dockerfile có 2 lệnh (instruction) là `FROM` và `COPY`;
"3 dòng" là cách nói gần đúng (2 lệnh cộng một dòng ghi chú `#`). Trong bài này mỗi lệnh có thêm một dòng comment
tiếng Việt phía trên nên file có 4 dòng, nhưng số lệnh thực thi vẫn đúng 2. Dòng comment (bắt đầu bằng `#`) không
phải lệnh và không tạo layer.

### 3.3. Giải thích từng lệnh

| Lệnh | Ý nghĩa |
|------|---------|
| `FROM nginx:alpine` | Lấy image `nginx` tag `alpine` làm nền. Image này đã có sẵn Nginx, cấu hình mặc định lắng nghe cổng 80 và `ENTRYPOINT`/`CMD` để tự chạy `nginx -g "daemon off;"`. Nhờ vậy Dockerfile không cần `CMD` hay `EXPOSE` riêng. |
| `COPY index.html /usr/share/nginx/html/index.html` | Copy file `index.html` từ build context (thư mục hiện tại) vào đường dẫn trong image. Đây là thư mục gốc phục vụ web mặc định của Nginx, ghi đè file `index.html` mẫu "Welcome to nginx!". Lệnh này tạo thêm đúng một layer mới. |

### 3.4. Vì sao chọn nginx:alpine

- Alpine Linux rất nhỏ (rootfs khoảng 9 MB, xem `docker history` ở mục 5.2) -> pull/push nhanh, ít gói phần mềm
  nên bề mặt tấn công nhỏ.
- Đo trên máy này: `docker image inspect nginx:alpine --format '{{.Size}}'` cho `26335715` byte (khoảng 26.3 MB).
- Số tham khảo (không đo trên máy này, thay đổi theo phiên bản): `nginx:latest` nền Debian cỡ khoảng 190 MB khi
  giải nén, lớn hơn nhiều lần. Với một trang HTML tĩnh thì bản Alpine là đủ.

## 4. Cấu trúc thư mục bài nộp

Thư mục `homework/session_09/ex3/` gồm: `Dockerfile`, `index.html`, `README.md` (báo cáo này),
`terminal_log.txt` (lệnh và output thật) và `.dockerignore`.

`.dockerignore` loại `README.md`, `terminal_log.txt` và `.git` khỏi build context. Lý do: `docker build .`
gửi toàn bộ thư mục hiện tại cho Docker daemon; các file tài liệu và thư mục `.git` không cần cho image.
Loại đi giúp context nhỏ gọn và tránh vô tình đưa file không liên quan vào image nếu sau này dùng `COPY . .`.
Các file build cần (`Dockerfile`, `index.html`) không bị loại.

## 5. Các bước thực hiện và kết quả

### 5.1. Chuẩn bị image nền

```
$ docker pull nginx:alpine
alpine: Pulling from library/nginx
e72112c14215: Pulling fs layer
...
e72112c14215: Pull complete
Digest: sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine

$ docker images nginx:alpine
IMAGE          ID             DISK USAGE   CONTENT SIZE   EXTRA
nginx:alpine   df221db836e1       94.4MB         27.2MB        

$ docker image inspect nginx:alpine --format '{{.Size}}'
26335715
```

(Dấu `...` lược bớt các dòng tiến trình trung gian; log đầy đủ trong `terminal_log.txt`.)

Lệnh `docker pull` được chạy riêng để log sạch (pull có thanh tiến trình); nếu bỏ qua, `docker build` cũng tự
pull ở bước `FROM`. Chạy lại lệnh pull về sau chỉ in `Image is up to date`.

### 5.2. Build image

Dùng `--progress=plain` để log không bị ghi đè dòng. Thêm `--no-cache` vì cache của các lần chạy thử trước đó
vẫn còn trên máy, nếu không bước `COPY` sẽ hiện `CACHED` và không minh họa được lần build đầu.
Lệnh theo đề bài là `docker build -t my-html-app:v1 .` (kết quả tương đương, chỉ khác cách hiển thị và cache).

```
$ docker build --no-cache --progress=plain -t my-html-app:v1 .
...
#4 [internal] load build context
#4 transferring context: 31B done
#4 DONE 0.0s

#5 [1/2] FROM docker.io/library/nginx:alpine@sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2
#5 resolve docker.io/library/nginx:alpine@sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2 0.0s done
#5 CACHED

#6 [2/2] COPY index.html /usr/share/nginx/html/index.html
#6 DONE 0.0s

#7 exporting to image
#7 exporting manifest sha256:97e4944a1521c395dcbce53d23a2459db6e928b83010fc740780659455a3c7e3 0.0s done
#7 exporting config sha256:b66bb4b743ae9345909a5f33976c0e3e968baa3ee46475e3594f7c18be7cfb06 0.0s done
#7 exporting attestation manifest sha256:e446dec4b0eff98e7610184489756a19c15fb28d161c92a159c711c0ce8c15c0 0.0s done
#7 exporting manifest list sha256:f9ce07f41b6582448b04e9b0dcc36ad9aeafce3f737eb785c51691143c2d0ca1 0.0s done
#7 naming to docker.io/library/my-html-app:v1 done
#7 DONE 0.2s
```

Đọc output: bước `[1/2] FROM` chỉ tham chiếu image nền có sẵn nên hiện `CACHED`; bước `[2/2] COPY` chạy thật
(`DONE`). Số bước build đúng bằng số lệnh trong Dockerfile.

```
$ docker images my-html-app
IMAGE            ID             DISK USAGE   CONTENT SIZE   EXTRA
my-html-app:v1   f9ce07f41b65       93.6MB         26.3MB        

$ docker history my-html-app:v1
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
f9ce07f41b65   1 second ago   COPY index.html /usr/share/nginx/html/index....   24.6kB    buildkit.dockerfile.v0
<missing>      2 weeks ago    RUN /bin/sh -c set -x     && apkArch="$(cat ...   52.4MB    buildkit.dockerfile.v0
<missing>      2 weeks ago    (...)  các dòng ENV/LABEL/STOPSIGNAL 0B được lược bớt
<missing>      2 weeks ago    CMD ["nginx" "-g" "daemon off;"]                0B        buildkit.dockerfile.v0
<missing>      2 weeks ago    EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      2 weeks ago    ENTRYPOINT ["/docker-entrypoint.sh"]            0B        buildkit.dockerfile.v0
<missing>      2 weeks ago    COPY 30-tune-worker-processes.sh /docker-ent...   16.4kB    buildkit.dockerfile.v0
<missing>      2 weeks ago    COPY 20-envsubst-on-templates.sh /docker-ent...   12.3kB    buildkit.dockerfile.v0
<missing>      2 weeks ago    COPY 15-local-resolvers.envsh /docker-entryp...   12.3kB    buildkit.dockerfile.v0
<missing>      2 weeks ago    COPY 10-listen-on-ipv6-by-default.sh /docker...   12.3kB    buildkit.dockerfile.v0
<missing>      2 weeks ago    COPY docker-entrypoint.sh / # buildkit          8.19kB    buildkit.dockerfile.v0
<missing>      2 weeks ago    RUN /bin/sh -c set -x     && addgroup -g 101...   5.72MB    buildkit.dockerfile.v0
<missing>      2 weeks ago    ENV NGINX_VERSION=1.31.6                        0B        buildkit.dockerfile.v0
<missing>      2 weeks ago    CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      2 weeks ago    ADD alpine-minirootfs-3.24.2-x86_64.tar.gz /...   9.08MB    buildkit.dockerfile.v0

$ docker image inspect my-html-app:v1 --format '{{.Id}}'
sha256:f9ce07f41b6582448b04e9b0dcc36ad9aeafce3f737eb785c51691143c2d0ca1
```

Nhận xét:

- Dòng trên cùng của `docker history` là layer do Dockerfile của bài tạo ra (`COPY`), chỉ `24.6kB` (phần lớn là
  phụ phí tar; nội dung file 34 byte). Các dòng `<missing>` bên dưới là layer thừa hưởng từ `nginx:alpine`.
- `DISK USAGE 93.6MB` là dung lượng giải nén trên đĩa; `CONTENT SIZE 26.3MB` là dung lượng nội dung image; hai số
  đo khác nhau nên không so trực tiếp. `Id` đổi khi build lại: xem mục 7.

### 5.3. Sơ đồ layer của image

```
my-html-app:v1
+------------------------------------------------------------+
| COPY index.html /usr/share/nginx/html/index.html           |  24.6kB   <- layer của bài này (trên cùng)
+============================================================+
| Các layer thừa hưởng từ nginx:alpine (read-only, dùng chung)|
|   RUN ... cài thêm gói (njs, acme ...)                     |  52.4MB
|   COPY 30-tune / 20-envsubst / 15-local / 10-listen / ...  |  vài chục kB (script entrypoint)
|   RUN ... addgroup, tạo user nginx, cài nginx              |  5.72MB
|   ENV, LABEL, EXPOSE 80, ENTRYPOINT, CMD ...               |  0B (chỉ là metadata)
|   ADD alpine-minirootfs-3.24.2-x86_64.tar.gz /             |  9.08MB   <- hệ điều hành Alpine (dưới cùng)
+------------------------------------------------------------+
        container đang chạy: thêm 1 lớp ghi (writable layer) mỏng bên trên
```

Layer của image chỉ đọc và được dùng chung giữa các image; container chạy thêm một lớp ghi mỏng ở trên cùng.

### 5.4. Chạy container

```
$ docker run -d -p 8081:80 --name html-app my-html-app:v1
01b6b813ce3c50f00f867cbb97f216714e9128d7b7642ae580e23bfdbe1d6000

$ docker ps --filter name=html-app
CONTAINER ID   IMAGE            COMMAND                  CREATED                  STATUS                  PORTS                                     NAMES
01b6b813ce3c   my-html-app:v1   "/docker-entrypoint...."   Less than a second ago   Up Less than a second   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   html-app

$ docker port html-app
80/tcp -> 0.0.0.0:8081
80/tcp -> [::]:8081
```

Ý nghĩa các tham số: `-d` chạy nền; `-p 8081:80` ánh xạ cổng 8081 của máy host tới cổng 80 trong container
(thứ tự là `host:container`); `--name html-app` đặt tên container.

```
$ docker logs html-app
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
...
/docker-entrypoint.sh: Configuration complete; ready for start up
(tiếp theo là các dòng [notice] của nginx: using the "epoll" event method, nginx/1.31.6,
 start worker process ...; đã rút gọn 16 dòng "start worker process", xem đầy đủ trong terminal_log.txt)
```

Các dòng đầu là script trong `/docker-entrypoint.d/` của image `nginx` chạy trước khi Nginx khởi động; dòng
`Configuration complete; ready for start up` cho biết Nginx đã sẵn sàng.

### 5.5. Kiểm tra theo đề bài

```
$ curl --no-progress-meter http://localhost:8081
<h1>Hello Docker Session 09!</h1>
```

Kết quả đúng chuỗi mong đợi. Lệnh theo đề là `curl http://localhost:8081`; trong log thật thêm
`--no-progress-meter` để bỏ bảng tiến trình của curl khi output đi qua pipe, nội dung trả về không đổi.

```
$ curl --no-progress-meter -I http://localhost:8081
HTTP/1.1 200 OK
Server: nginx/1.31.6
...
Content-Type: text/html
Content-Length: 34
...
Accept-Ranges: bytes

$ curl -s http://localhost:8081 | diff - index.html && echo "GIONG HET"
GIONG HET
```

`Content-Length: 34` khớp kích thước file (33 ký tự + 1 xuống dòng); `diff` không báo khác biệt.

```
$ docker exec html-app cat /usr/share/nginx/html/index.html
<h1>Hello Docker Session 09!</h1>

$ docker exec html-app ls -la /usr/share/nginx/html/
total 16
drwxr-xr-x    1 root     root          4096 Oct  8 16:51 .
drwxr-xr-x    1 root     root          4096 Sep 22 21:19 ..
-rw-r--r--    1 root     root           497 Sep 15 14:18 50x.html
-rwxr-xr-x    1 root     root            34 Oct  8 16:50 index.html
```

Thư mục có hai file: `50x.html` (trang lỗi mặc định của image nền) và `index.html` do ta copy vào (34 byte).

## 6. Layer caching

Docker lưu kết quả từng bước build. Khi build lại, nếu lệnh và dữ liệu đầu vào của một bước không đổi thì
bước đó dùng lại kết quả cũ (`CACHED`). Hễ một bước thay đổi thì mọi bước phía sau nó đều phải chạy lại.

Thử nghiệm: thêm một dòng `<p>` vào `index.html`, build thành tag `my-html-app:v2`.

```
$ echo '<p>Phien ban v2 - kiem tra layer cache</p>' >> index.html

$ cat index.html
<h1>Hello Docker Session 09!</h1>
<p>Phien ban v2 - kiem tra layer cache</p>

$ docker build --progress=plain -t my-html-app:v2 .
...
#4 [internal] load build context
#4 transferring context: 114B done
#4 DONE 0.0s

#5 [1/2] FROM docker.io/library/nginx:alpine@sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2
#5 CACHED

#6 [2/2] COPY index.html /usr/share/nginx/html/index.html
#6 DONE 0.0s
...
#7 naming to docker.io/library/my-html-app:v2 done

$ docker images my-html-app
IMAGE            ID             DISK USAGE   CONTENT SIZE   EXTRA
my-html-app:v1   f9ce07f41b65       93.6MB         26.3MB   U    
my-html-app:v2   2550676d1bd9       93.6MB         26.3MB        
```

Dấu `...` là các dòng log được lược bớt (bản đầy đủ trong `terminal_log.txt`).

Nhận xét:

- Bước `[1/2]Nhận xét:

- Bước `[1/2] FROM` là `CACHED` (không đổi), bước `[2/2] COPY` chạy lại (`DONE`) vì nội dung `index.html` đã đổi
  (build context tăng từ 71B lên 114B).
- Hai tag cùng hiển thị `93.6MB` / `26.3MB`: chúng dùng chung toàn bộ layer của `nginx:alpine`, chỉ khác
  nhau layer `COPY` vài chục kB. Có thêm tag không làm kích thước tăng gấp đôi.
- Ý nghĩa thực tế: đặt lệnh ít thay đổi (cài gói) lên trên, lệnh hay thay đổi (copy mã nguồn) xuống dưới để tận dụng cache.

Khôi phục `index.html` về đúng một dòng của đề, build lại `v1` (mọi bước đã có sẵn trong cache):

```
$ printf '<h1>Hello Docker Session 09!</h1>\n' > index.html

$ docker build --progress=plain -t my-html-app:v1 .
...
#5 [1/2] FROM docker.io/library/nginx:alpine@sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2
#5 DONE 0.0s

#6 [2/2] COPY index.html /usr/share/nginx/html/index.html
#6 CACHED

#7 exporting manifest sha256:97e4944a1521c395dcbce53d23a2459db6e928b83010fc740780659455a3c7e3 done
#7 exporting config sha256:b66bb4b743ae9345909a5f33976c0e3e968baa3ee46475e3594f7c18be7cfb06 done
...
#7 naming to docker.io/library/my-html-app:v1 done

$ docker rmi my-html-app:v2
Untagged: my-html-app:v2
Deleted: sha256:2550676d1bd986e97c4059eff4437be3151d34d0d30f07bca415ec7cf701a549
```

Lần build lại này bước `COPY` là `CACHED` vì nội dung `index.html` giống hệt lần build đầu. Mã `exporting manifest`
và `exporting config` giống hệt lần build đầu (`97e4944a...` và `b66bb4b7...`): image không đổi về nội dung.

## 7. Bảng lệnh Docker thường dùng

| Nhóm | Lệnh | Tác dụng |
|------|------|----------|
| Image | `docker images` / `docker images my-html-app` | Liệt kê image (lọc theo tên) |
| Image | `docker build -t ten:tag .` | Build image từ Dockerfile ở thư mục hiện tại (dấu `.` là build context) |
| Image | `docker history ten:tag` | Xem các layer của image và kích thước |
| Image | `docker rmi ten:tag` | Xóa tag/image (không xóa được nếu đang có container dùng) |
| Container | `docker run -d -p 8081:80 --name html-app my-html-app:v1` | Tạo và chạy container nền, map cổng |
| Container | `docker ps` / `docker ps -a` | Liệt kê container đang chạy / tất cả |
| Container | `docker logs html-app` (thêm `-f` để theo dõi) | Xem log của container |
| Container | `docker exec html-app ls /usr/share/nginx/html` | Chạy lệnh trong container đang chạy (`-it ... sh` để vào shell) |
| Container | `docker stop html-app` | Dừng container |
| Container | `docker rm html-app` (`-f` để xóa cả khi đang chạy) | Xóa container |

Về `Id`: build lần đầu cho `sha256:f9ce07f41b65...`; sau khi build lại `v1` ở mục 6, `Id` là
`sha256:6c1a9a8affee6417f67562797a0b1481b069c5a280049eae918edc31abbe4435` dù manifest và config không đổi,
vì `Id` là digest của manifest list (đổi theo attestation).

## 8. Xử lý sự cố thường gặp

Các thông báo dưới đây là dạng thường gặp (tham khảo, không tái hiện trong bài này; riêng thông báo
`Conflict` đúng với nội dung đề bài). Lỗi đường dẫn của Git Bash ở dòng cuối đã gặp thật trong lần chạy đầu.

| Sự cố | Thông báo thường gặp | Nguyên nhân và cách xử lý |
|-------|----------------------|----------------------------|
| Cổng 8081 đang bận | `Bind for 0.0.0.0:8081 failed: port is already allocated` | Container hoặc tiến trình khác đang dùng cổng 8081. Tìm bằng `docker ps` (container) hoặc `netstat -ano \| findstr :8081` (Windows). Dừng tiến trình đó, hoặc map cổng khác: `-p 8082:80`. |
| Trùng tên container | `Conflict. The container name "/html-app" is already in use` | Đã có container tên `html-app` (kể cả đã dừng). Xóa bằng `docker rm -f html-app` rồi chạy lại, hoặc đặt tên khác. Kiểm tra bằng `docker ps -a`. |
| Quên dấu `.` ở cuối `docker build` | `docker: 'docker buildx build' requires 1 argument` (tùy phiên bản) | Lệnh build cần tham số build context. Dùng `docker build -t my-html-app:v1 .` (dấu chấm = thư mục hiện tại). |
| Docker daemon chưa chạy | `failed to connect to the docker API ... check if the path is correct and if the daemon is running` | Mở Docker Desktop, đợi trạng thái chạy, thử lại `docker info`. |
| `docker exec` báo đường dẫn `C:/Program Files/Git/...` | `cat: can't open 'C:/Program Files/Git/usr/share/nginx/html/index.html'` | Chỉ trên Git Bash: tự đổi đường dẫn bắt đầu bằng `/`. Dùng `MSYS_NO_PATHCONV=1` hoặc `//usr/share/...`. |

## 9. Đối chiếu yêu cầu đề bài

| Yêu cầu | Thực hiện | Bằng chứng |
|---------|-----------|------------|
| Tạo `index.html` chứa `<h1>Hello Docker Session 09!</h1>` | Đã tạo, đúng một dòng | Mục 3.1 |
| Dockerfile gồm `FROM nginx:alpine` và `COPY index.html /usr/share/nginx/html/index.html` | Đã viết đúng 2 lệnh (có thêm comment `#`) | Mục 3.2 |
| Build image tên `my-html-app:v1` | Đã build | Mục 5.2, `docker images my-html-app` |
| Chạy container trên cổng 8081 | `docker run -d -p 8081:80 --name html-app my-html-app:v1` | Mục 5.4, `docker ps`, `docker port` |
| `curl http://localhost:8081` trả về đúng chuỗi | Trả về `<h1>Hello Docker Session 09!</h1>` | Mục 5.5 |
| Nộp `index.html`, `Dockerfile`, `README.md` tại `homework/session_09/ex3/` | Đã có, kèm `terminal_log.txt` và `.dockerignore` | Mục 4 |
| Dọn dẹp | Đã `docker stop`/`docker rm` container; giữ lại image `my-html-app:v1` | Mục 10 |

## 10. Dọn dẹp

```
$ docker stop html-app
html-app

$ docker rm html-app
html-app

$ docker ps -a --filter name=html-app
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker images my-html-app
IMAGE            ID             DISK USAGE   CONTENT SIZE   EXTRA
my-html-app:v1   6c1a9a8affee       93.6MB         26.3MB        
```

Container `html-app` đã xóa (danh sách rỗng), tag `v2` đã xóa ở mục 6, image `my-html-app:v1` được giữ lại để kiểm tra.

## 11. Kết luận

- Chỉ với hai lệnh `FROM` và `COPY` đã đóng gói được một trang web tĩnh thành image chạy được: image nền
  `nginx:alpine` lo phần web server, Dockerfile chỉ cần đặt đúng file vào `/usr/share/nginx/html/`.
- Image có kích thước nội dung khoảng 26.3 MB; layer riêng của bài chỉ khoảng 24.6 kB. `curl` trả về đúng
  `<h1>Hello Docker Session 09!</h1>`, `diff` với file gốc giống hệt.
- Layer caching: chỉ bước có thay đổi (`COPY`) chạy lại, các bước trước dùng cache.
- Điểm cần nhớ: map cổng theo thứ tự `host:container`, đặt tên container để dễ quản lý, dọn container sau khi dùng
  và lưu ý lỗi đường dẫn của Git Bash khi dùng `docker exec` trên Windows.

# Hybrid Chat (Client-Server + P2P) – Assignment 1

Ứng dụng chat lai (hybrid) kết hợp:

- **Client-Server**: dùng tracker để đăng ký peer, khám phá peer, quản lý channel.
- **Peer-to-Peer (P2P)**: nhắn tin trực tiếp giữa các peer khi đang chat.

UI chạy bằng WeApRous + HTML/JS, hỗ trợ:

- Danh sách peer online
- Connect peer
- Direct message (1-1)
- Broadcast đến các peer đã kết nối
- Join channel + gửi message theo channel

---

## 1) Cấu trúc chạy chính

- `start_tracker_http.py`: tracker HTTP (đăng ký peer, danh sách peer, channel)
- `start_client.py`: 1 peer chat (webapp + API peer)
- `start_chat.py`: script demo nhanh (tự bật tracker + 3 peer)

---

## 2) Yêu cầu môi trường

- Python 3.8+ (khuyến nghị)
- Chạy trong thư mục gốc project
- Windows/Linux đều chạy được cho phần chat demo

---

## 3) Chạy demo nhanh nhất

```bash
python start_chat.py
```

Script này sẽ tự khởi tạo:

- Tracker tại `localhost:9998`
- 3 peer:
  - `alice` tại `localhost:9000`
  - `bob` tại `localhost:9001`
  - `robo` tại `localhost:9002`

Mở trình duyệt:

- `http://localhost:9000/chat.html`
- `http://localhost:9001/chat.html`
- `http://localhost:9002/chat.html`

Dừng demo: nhấn `Ctrl + C` ở terminal chạy `start_chat.py`.

---

## 4) Chạy thủ công từng tiến trình (nếu cần)

### 4.1 Chạy tracker

```bash
python start_tracker_http.py --host 0.0.0.0 --port 9998
```

### 4.2 Chạy từng peer

```bash
python start_client.py --addr 0.0.0.0:9000 --username alice --tracker 0.0.0.0:9998
python start_client.py --addr 0.0.0.0:9001 --username bob --tracker 0.0.0.0:9998
```

Mỗi peer có UI tại:

- `http://localhost:<port>/chat.html`

---

## 5) Luồng demo đề xuất (để thuyết trình)

1. Mở 2 tab/browser cho 2 peer khác nhau (ví dụ 9000 và 9001).
2. Quan sát danh sách online peers.
3. Peer A bấm **Connect** tới Peer B.
4. Gửi **Direct Message** (1-1) từ A sang B.
5. Gửi **Broadcast** từ A để B nhận.
6. Cả A và B **Join** cùng một channel (ví dụ `general`).
7. A gửi **Send to channel**, B nhận được tin kèm tên channel.

---

## 6) API chính

### Tracker (`start_tracker_http.py`)

- `POST /register`
  - body: `{ "ip": "...", "port": "...", "username": "..." }`
- `GET /peers`
  - trả danh sách peer đã đăng ký
- `GET /listchannels`
  - trả danh sách channel + thành viên channel
- `POST /joinchannel`
  - body: `{ "addr": "ip:port", "channel": "general" }`

### Peer (`start_client.py`)

- `GET /list` → lấy peer list từ tracker
- `POST /connectpeer` → connect P2P với peer khác
- `POST /send` → direct message 1-1
- `POST /broadcast` → gửi đến peers đã kết nối
- `POST /inbox` + `GET /pollinbox` → nhận/poll message
- `GET /listchannels` + `POST /joinchannel` → thao tác channel
- `POST /sendchannel` → gửi message channel (client hỏi tracker để lấy thành viên kênh, sau đó gửi P2P)

---

## 7) Giao diện hiện tại

UI chat đã được chỉnh để dễ dùng hơn:

- Nhắn riêng không cần nhập tay địa chỉ người nhận.
- Có thể chọn peer trực tiếp từ danh sách/ô chọn người nhận.
- Vẫn hỗ trợ đầy đủ direct message, broadcast, channel message.

---

## 8) Không dùng tracker

Có thể chạy:

```bash
python start_chat.py --no-tracker
```

Khi đó peer vẫn chạy, nhưng sẽ hạn chế:

- Không có danh sách peer online từ tracker
- Không có channel management qua tracker

---

## 9) Ghi chú kỹ thuật

- Mô hình hiện tại ưu tiên demo học phần: rõ luồng tracker + P2P, dễ quan sát.
- Channel message hiện theo hướng: peer gửi trực tiếp đến các peer cùng channel (sau khi truy vấn tracker).
- Nếu cần mở rộng production: thêm auth, retry, persistent storage, heartbeat, và kiểm soát truy cập channel.

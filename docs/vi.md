**[English](../README.md)** | **[Tiếng Việt](vi.md)**

![Cloudflare Gateway](https://github.com/luxysiv/Cloudflare-Gateway-Pihole/assets/46205571/b8b7b12b-2fd8-4978-8e3c-2472a4167acb)

# Cloudflare Gateway DNS Filter — Chặn + Cho phép

> Chặn quảng cáo theo kiểu Pihole bằng Cloudflare Gateway Zero Trust, kết hợp rule Allow ưu tiên cao hơn rule Block — tất cả trong một workflow duy nhất.

---

## Cơ chế hoạt động

Script quản lý hai tập rule DNS trên Cloudflare Gateway trong một lần chạy duy nhất:

| Rule | Hành động | Độ ưu tiên | Nguồn dữ liệu |
| :--- | :--- | :--- | :--- |
| `[AdBlock-DNS-Filters] Block Ads` | block | 1000 | `adlist.ini` + `dynamic_blacklist.txt` |
| `[AdAllow-DNS-Filters] Allow` | allow | 999 *(ưu tiên cao hơn)* | `whitelist.ini` + `dynamic_whitelist.txt` |

Vì rule Allow có **số độ ưu tiên thấp hơn** (999 < 1000), Cloudflare đánh giá nó trước — tên miền được cho phép luôn được phân giải kể cả khi nó nằm trong danh sách chặn. Script không cần trừ whitelist khỏi blocklist nữa.

Script cũng kiểm tra **giới hạn 300 lists** của gói miễn phí Cloudflare Gateway (tính tổng cả block lẫn allow). Nếu nguồn dữ liệu của bạn cần nhiều hơn 300 lists, script dừng lại trước khi gọi bất kỳ API nào.

---

## Hướng dẫn cài đặt

### 1. Fork repository này về tài khoản của bạn

### 2. Lấy thông tin xác thực Cloudflare

- **Account ID** — lấy từ URL sau `https://dash.cloudflare.com/`:
  `https://dash.cloudflare.com/?to=/:account/workers`

- **API Token** — tạo tại `https://dash.cloudflare.com/profile/api-tokens` với 3 quyền sau:
  1. `Account › Zero Trust : Edit`
  2. `Account › Account Firewall Access Rules : Edit`
  3. `Account › Access: Apps and Policies : Edit`

Tạo `CF_API_TOKEN` như sau:

![CF_API_TOKEN](https://github.com/luxysiv/Cloudflare-Gateway-Pihole/assets/46205571/a5b90438-26cc-49ae-9a55-5409a90b683f)

### 3. Thêm Repository Secrets

Vào `https://github.com/<username>/Cloudflare-Gateway-DNS-Filter/settings/secrets/actions` và thêm:

| Secret | Giá trị |
| :--- | :--- |
| `CF_API_TOKEN` | API Token vừa tạo |
| `CF_IDENTIFIER` | Account ID của bạn |

Secret Github Action như sau:

![1000015672](https://github.com/luxysiv/Cloudflare-Gateway-Pihole/assets/46205571/6bd7f41d-0ca5-4944-95d3-d41dfd913c60)

### 4. Cấu hình danh sách

**Danh sách chặn** — chỉnh sửa [`lists/adlist.ini`](../lists/adlist.ini):
```ini
[Ad-Urls]
Adguard = https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
```

**Danh sách cho phép** — chỉnh sửa [`lists/whitelist.ini`](../lists/whitelist.ini):
```ini
[Allow-Urls]
MyAllow = https://example.com/my-whitelist.txt
```

Cả hai file đều chấp nhận URL thuần (mỗi dòng một URL) hoặc định dạng INI `[Section] key = url`.

### 5. (Tuỳ chọn) Thêm danh sách qua GitHub Actions Variables

Vào `https://github.com/<username>/Cloudflare-Gateway-DNS-Filter/settings/variables/actions` và thêm:

| Name | Value |
| :--- | :--- |
| `ADLIST_URLS` | Các URL blocklist cách nhau bằng dấu cách |
| `WHITELIST_URLS` | Các URL allowlist cách nhau bằng dấu cách |
| `DYNAMIC_BLACKLIST` | Tên miền chặn thêm, mỗi dòng một tên |
| `DYNAMIC_WHITELIST` | Tên miền cho phép thêm, mỗi dòng một tên |

Dùng Variables giúp danh sách tuỳ chỉnh của bạn không bị mất khi pull cập nhật repo.

Bạn cũng có thể chỉnh trực tiếp các file local:
- [`lists/dynamic_blacklist.txt`](../lists/dynamic_blacklist.txt)
- [`lists/dynamic_whitelist.txt`](../lists/dynamic_whitelist.txt)

---

## Định dạng danh sách được hỗ trợ

```
# URL thuần
https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
```
```ini
# Định dạng INI
[Ad-Urls]
Adguard = https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
```

Hỗ trợ các định dạng: hosts file, cú pháp AdBlock/uBlock, danh sách domain thuần.

> **Lưu ý dành cho người Việt Nam:** Dùng **bộ lọc DNS**, không phải bộ lọc cho browser extension. Bộ lọc browser chứa các quy tắc CSS/cosmetic sẽ gây lỗi khi dùng làm DNS blocklist.

---

## Chạy trên máy cá nhân (Termux hoặc terminal bất kỳ)

### Cách 1 — Clone từ GitHub

```sh
# Cài đặt công cụ cần thiết
yes | pkg upgrade
yes | pkg install python-pip git

# Clone repo của bạn
git clone https://github.com/<username>/Cloudflare-Gateway-DNS-Filter.git
cd Cloudflare-Gateway-DNS-Filter

# Chỉnh thông tin xác thực
nano .env
```

Chạy đồng bộ:
```sh
python -m src run
```

Xoá toàn bộ lists và rules đã tạo:
```sh
python -m src leave
```

### Cách 2 — Tải ZIP

1. Tải file ZIP từ trang GitHub → **Code → Download ZIP**.
2. Giải nén và chỉnh `.env`, `lists/adlist.ini`, `lists/whitelist.ini`.
3. Mở Termux:

```sh
yes | pkg upgrade
yes | pkg install python-pip
termux-setup-storage
cd storage/downloads/Cloudflare-Gateway-DNS-Filter-main
python -m src run
```

---

## Lịch chạy tự động

Workflow tự động chạy vào **thứ Sáu hàng tuần lúc 0:00 UTC** và mỗi lần push. Bạn cũng có thể kích hoạt thủ công từ tab **Actions**.

GitHub tự động vô hiệu hoá Actions sau 60 ngày không có push. Để tránh điều này, dùng Cloudflare Workers để tự động kích hoạt workflow:

```javascript
addEventListener('scheduled', event => {
  event.waitUntil(handleScheduledEvent());
});

async function handleScheduledEvent() {
  const GITHUB_TOKEN = 'YOUR_GITHUB_TOKEN';   // cần quyền workflow, không hết hạn
  const GITHUB_USER  = 'YOUR_USERNAME';
  const GITHUB_REPO  = 'YOUR_REPO_NAME';
  const WORKFLOW_ID  = 'main.yml';

  const url = `https://api.github.com/repos/${GITHUB_USER}/${GITHUB_REPO}/actions/workflows/${WORKFLOW_ID}/dispatches`;

  const res = await fetch(url, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${GITHUB_TOKEN}`,
      'Content-Type': 'application/json',
      'User-Agent': 'Cloudflare-Worker-Trigger',
    },
    body: JSON.stringify({ ref: 'main' }),
  });

  if (!res.ok) throw new Error(`Dispatch thất bại: ${res.status}`);
  console.log('Đã kích hoạt GitHub Action thành công');
}
```

Nhớ cài **Cron Trigger** trong cài đặt Cloudflare Worker (ví dụ: `0 0 * * 5`).

---

## Xoá toàn bộ tài nguyên đã tạo

Đổi lệnh trong bước workflow thành `leave` rồi chạy một lần:

```yml
- name: Cloudflare Gateway Zero Trust
  run: python -m src leave
```

Script sẽ xoá tất cả lists và rules do script này tạo ra khỏi tài khoản Cloudflare của bạn.

---

## Giới hạn

| Giới hạn | Giá trị |
| :--- | :--- |
| Số domain mỗi list | 1.000 |
| Tổng số lists (block + allow cộng lại) | ≤ 300 (gói miễn phí) |
| Tổng số domain (300 × 1.000) | ≤ 300.000 |

Script tính số lists cần thiết trước khi gọi bất kỳ API nào và dừng với thông báo lỗi rõ ràng nếu vượt giới hạn.

---

👌 Chúc các bạn thành công!

👌 Mọi thắc mắc về script, các bạn có thể mở issue.

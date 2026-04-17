# Hermes Employee Agent — Hướng dẫn deploy lên OPKMP

Hướng dẫn đầy đủ để deploy một Hermes Employee Agent công khai và list lên Open Knowledge Marketplace.

---

## Bước 1: Chuẩn bị VPS

### Spec tối thiểu

```
Hệ điều hành : Ubuntu 22.04 LTS trở lên
RAM          : 2GB (khuyến nghị 4GB nếu dùng STT local)
Disk         : 20GB
CPU          : 1 vCPU (đủ cho agent text-only)
```

Các nhà cung cấp phổ biến: DigitalOcean, Vultr, Linode, hoặc bất kỳ VPS nào có Ubuntu 22.04.

### Mở port cần thiết

Hermes gateway không cần port mở ra internet — nó dùng outbound long-polling với Telegram.
Bạn chỉ cần đảm bảo VPS có kết nối internet outbound bình thường.

```bash
# Kiểm tra kết nối outbound (phải pass)
curl -s https://api.telegram.org > /dev/null && echo "OK: Telegram reachable"
curl -s https://api.anthropic.com > /dev/null && echo "OK: Anthropic reachable"
```

Nếu dùng firewall (ufw), không cần thêm rule gì đặc biệt cho Telegram polling mode.

---

## Bước 2: Cài Hermes Agent

### Cài đặt

SSH vào VPS của bạn, chạy lệnh cài Hermes:

```bash
curl -fsSL https://hermes.nousresearch.com/install | bash
```

Sau khi cài xong, restart shell hoặc chạy:

```bash
source ~/.bashrc
```

Kiểm tra cài thành công:

```bash
hermes --version
```

### Cấu hình LLM provider

Chạy lệnh setup để nhập API key và chọn model:

```bash
hermes setup
```

Hermes sẽ hỏi:
- Provider (ví dụ: anthropic, openrouter, azure)
- API key
- Model mặc định (ví dụ: claude-sonnet-4-5, gpt-4o, ...)

Sau khi setup, config được lưu tại `~/.hermes/config.yaml`.

---

## Bước 3: Cấu hình Public Mode

Config mặc định của Hermes là **Personal Mode** — có memory, honcho, full tools.
Để dùng làm Employee Agent, bạn phải chuyển sang **Public Mode**.

### Copy template config

```bash
cp /path/to/hermes-employee-guide/templates/config.yaml ~/.hermes/config.yaml
```

Hoặc tải về trực tiếp:

```bash
curl -fsSL https://raw.githubusercontent.com/mrqhocungdungai-vn/hermes-employee-guide/main/templates/config.yaml \
  -o ~/.hermes/config.yaml
```

### Các field bắt buộc phải thay đổi

Mở `~/.hermes/config.yaml` và điền vào các placeholder:

```bash
nano ~/.hermes/config.yaml
```

**Phần model — chọn provider và model bạn dùng:**

```yaml
model:
  default: YOUR_MODEL_NAME          # ví dụ: claude-sonnet-4-5
  provider: YOUR_PROVIDER           # ví dụ: anthropic, openrouter
```

**Phần system_prompt — knowledge base của agent:**

```yaml
agent:
  system_prompt: |
    # [Tên Agent] — Knowledge Base

    ## Bạn là ai
    [Viết 2-3 câu mô tả agent, phục vụ cho ai, chuyên về gì]

    ## Nhiệm vụ
    [Liệt kê 3-5 việc agent được phép làm]

    ## Giới hạn
    [Liệt kê những gì agent KHÔNG làm]
```

**Tại sao memory_enabled: false?**

Khi nhiều người dùng chat cùng lúc, memory của người này không được lẫn vào người kia.
Tắt memory đảm bảo mỗi session độc lập — đây là yêu cầu bắt buộc cho public agent.

```yaml
memory:
  memory_enabled: false         # BẮT BUỘC tắt cho public mode
  user_profile_enabled: false   # BẮT BUỘC tắt cho public mode
```

**Tại sao group_sessions_per_user: true?**

Field này đảm bảo mỗi Telegram user_id có session riêng biệt — người A chat không thấy
lịch sử của người B. Không có setting này, nhiều user có thể share chung 1 session.

```yaml
group_sessions_per_user: true   # BẮT BUỘC cho multi-user
```

**Tại sao platform_toolsets telegram: [safe]?**

Toolset "safe" loại bỏ hoàn toàn terminal, file, process khỏi agent — ngay cả khi có ai
cố tình yêu cầu agent chạy lệnh hệ thống, agent không có tool đó để chạy.
Đây là lớp bảo vệ đầu tiên (layer 1).

```yaml
platform_toolsets:
  telegram:
    - safe    # chỉ web + vision — không có terminal, file, process
```

**Tại sao honcho phải tắt?**

Honcho là memory provider nội bộ — không phù hợp cho public agent vì lưu dữ liệu user.
Để trống hoặc tắt hoàn toàn:

```yaml
memory:
  provider: ""    # tắt honcho
```

---

## Bước 4: Viết SOUL.md cho agent của bạn

### SOUL.md là gì?

SOUL.md là file định nghĩa **identity** của agent — tên, vai trò, tính cách, giới hạn.
Khác với `system_prompt` (chứa knowledge/dữ liệu), SOUL.md định nghĩa agent *là ai* và
*hành xử như thế nào*.

Hermes tự động đọc SOUL.md từ `~/.hermes/SOUL.md` khi khởi động.

### Copy và điền template

```bash
cp /path/to/hermes-employee-guide/templates/SOUL.md ~/.hermes/SOUL.md
nano ~/.hermes/SOUL.md
```

### Hướng dẫn điền từng phần

**Phần Identity — trả lời câu hỏi "Bạn là ai?":**

Agent cần trả lời được câu này một cách nhất quán. Viết rõ tên, vai trò, và quan trọng
nhất là nói rõ agent KHÔNG phải ChatGPT hay AI đa năng.

**Phần Nhiệm vụ cốt lõi — 3-5 điểm, càng cụ thể càng tốt:**

Đừng viết chung chung "hỗ trợ khách hàng". Viết cụ thể:
- "Trả lời câu hỏi về sản phẩm X, Y, Z"
- "Hướng dẫn quy trình đặt hàng"
- "Giải thích chính sách hoàn tiền"

**Phần Giới hạn — quan trọng không kém nhiệm vụ:**

Viết rõ những gì agent KHÔNG làm. Ví dụ:
- Không tư vấn pháp lý, y tế
- Không tiết lộ thông tin nội bộ
- Không thực hiện giao dịch tài chính

**Phần Escalation — khi nào chuyển sang người thật:**

Định nghĩa điều kiện escalate và cách thức (link, số điện thoại, email...).

---

## Bước 5: Cài và cấu hình AgentShield

AgentShield là lớp bảo vệ thứ hai — rate limiting, topic guard, chặn lệnh hệ thống.
Bắt buộc cho bất kỳ agent nào chạy public.

### Cài đặt

```bash
git clone https://github.com/mrqhocungdungai-vn/agentshield.git ~/agentshield
cd ~/agentshield
bash install.sh
```

Script install.sh sẽ:
- Copy hook files vào `~/.hermes/hooks/agentshield/`
- Copy config mẫu vào `~/.hermes/agentshield.yaml`

### Cấu hình qua TUI

```bash
agentshield-config
```

Giao diện TUI sẽ hỏi bạn:

**Rate limiting:**
- Messages per minute: khuyến nghị 10 (đủ cho cuộc trò chuyện bình thường)
- Messages per day: khuyến nghị 100-200

**Topic guard:**
- Thêm các từ khóa cần chặn (ví dụ: "api key", "password", "sudo", "jailbreak")
- Block message: thông báo trả về khi bị chặn

Hoặc chỉnh sửa trực tiếp `~/.hermes/agentshield.yaml` — xem template tại
`templates/agentshield.yaml` trong repo này.

### Bật TELEGRAM_ALLOW_ALL_USERS

Đây là bước hay bị quên nhất. Hermes có lớp auth riêng chạy TRƯỚC AgentShield.
Nếu không bật, người lạ sẽ thấy thông báo "pairing" thay vì đến được AgentShield.

```bash
echo "TELEGRAM_ALLOW_ALL_USERS=true" >> ~/.hermes/.env
```

Lý do: khi set `TELEGRAM_ALLOW_ALL_USERS=true`, Hermes bỏ qua lớp auth riêng của nó
và để AgentShield kiểm soát toàn bộ access. Đây là cách đúng khi dùng AgentShield.

---

## Bước 6: Khởi động gateway

### Chạy gateway

```bash
hermes gateway run
```

Lần đầu chạy, Hermes sẽ hỏi cấu hình platform (Telegram, Discord, ...).
Xem Bước 7 để kết nối Telegram.

### Chạy dưới dạng service (khuyến nghị cho production)

Để agent chạy liên tục kể cả khi bạn đóng SSH:

```bash
# Tạo systemd user service
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/hermes-gateway.service << 'EOF'
[Unit]
Description=Hermes Gateway
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/hermes gateway run
Restart=on-failure
RestartSec=5
WorkingDirectory=%h

[Install]
WantedBy=default.target
EOF

# Kích hoạt và khởi động
systemctl --user daemon-reload
systemctl --user enable hermes-gateway
systemctl --user start hermes-gateway

# Kiểm tra trạng thái
systemctl --user status hermes-gateway
```

### Kiểm tra agent đang chạy

```bash
# Xem log realtime
journalctl --user -u hermes-gateway -f

# Kiểm tra AgentShield đã load chưa
journalctl --user -u hermes-gateway -n 30 --no-pager | grep -i "hook\|agentshield"
# Kết quả mong đợi: [hooks] Loaded hook 'agentshield' for events: ['before_message', 'agent:end']
```

---

## Bước 7: Kết nối platform (ví dụ Telegram)

### Tạo Telegram Bot

1. Mở Telegram, tìm kiếm **@BotFather**
2. Gửi lệnh `/newbot`
3. Đặt tên bot (ví dụ: "MRQ Support Bot")
4. Đặt username (phải kết thúc bằng `bot`, ví dụ: `mrqsupport_bot`)
5. BotFather trả về **Bot Token** — dạng `123456789:ABCDEFxxxxx`

### Thêm token vào config Hermes

```bash
# Cách 1: qua file .env
echo "TELEGRAM_BOT_TOKEN=YOUR_BOT_TOKEN_HERE" >> ~/.hermes/.env

# Cách 2: qua hermes setup
hermes setup
# Chọn platform Telegram, nhập token khi được hỏi
```

### Restart gateway

```bash
systemctl --user restart hermes-gateway
# hoặc nếu chạy tay: Ctrl+C rồi chạy lại hermes gateway run
```

### Test gửi tin nhắn

1. Mở Telegram, tìm bot của bạn theo username
2. Gửi `/start`
3. Gửi một câu hỏi thử

Nếu bot phản hồi đúng — agent đang hoạt động.

**Kiểm tra rate limit hoạt động:**
Gửi 11 tin nhắn liên tiếp trong 1 phút — tin thứ 11 phải nhận được thông báo rate limit.

**Kiểm tra topic guard:**
Gửi tin nhắn chứa từ "jailbreak" hoặc "api key" — phải nhận được block message.

---

## Bước 8: List lên OPKMP

### Truy cập Open Knowledge Marketplace

1. Vào https://www.openknowledgemarket.com
2. Đăng ký tài khoản creator (nếu chưa có)

### Submit agent

Điền form submit với các thông tin:

**Tên agent:**
Tên rõ ràng, mô tả được chức năng. Ví dụ: "MRQ AI Support — Tư vấn AI Tools"

**Mô tả:**
2-3 câu ngắn gọn: agent làm gì, phục vụ ai, điểm khác biệt là gì.

**Link kết nối:**
Đây là link người dùng dùng để chat với agent của bạn.
- Telegram bot: `https://t.me/your_bot_username`
- Hoặc endpoint API nếu bạn expose qua web

**Danh mục:**
Chọn danh mục phù hợp (ví dụ: Customer Support, Education, Business...)

**Giá:**
OPKMP hỗ trợ free và paid. Nếu free, để $0. Nếu paid, set giá bạn muốn.

### Done

Sau khi submit, agent của bạn sẽ xuất hiện trên marketplace.
Người dùng click link → chat với bot Telegram của bạn → bạn nhận được traffic.

---

## Troubleshooting nhanh

**Bot không phản hồi:**
```bash
# Kiểm tra gateway có đang chạy không
systemctl --user status hermes-gateway

# Kiểm tra log lỗi
journalctl --user -u hermes-gateway -n 50 --no-pager | grep -i "error\|ERROR"

# Kiểm tra token đúng chưa
grep TELEGRAM ~/.hermes/.env
```

**Người lạ thấy "pairing message" thay vì chat được:**
```bash
# Phải có dòng này trong .env
grep TELEGRAM_ALLOW_ALL ~/.hermes/.env
# Nếu không có -> thêm vào:
echo "TELEGRAM_ALLOW_ALL_USERS=true" >> ~/.hermes/.env
systemctl --user restart hermes-gateway
```

**AgentShield không load:**
```bash
# Kiểm tra hook files có đúng chỗ không
ls ~/.hermes/hooks/agentshield/
# Phải có: HOOK.yaml và handler.py

# Kiểm tra agentshield.yaml hợp lệ
python3 -c "import yaml; yaml.safe_load(open('$HOME/.hermes/agentshield.yaml'))" && echo "YAML OK"
```

**Bot trả lời sai hoặc ngoài scope:**
Cập nhật lại `system_prompt` trong `~/.hermes/config.yaml` và restart gateway.
Thêm ví dụ cụ thể hơn về những gì agent KHÔNG làm.

---

## Tài liệu liên quan

- Hermes Agent docs: https://github.com/NousResearch/hermes-agent
- AgentShield repo: https://github.com/mrqhocungdungai-vn/agentshield
- OPKMP: https://www.openknowledgemarket.com
- MRQ TikTok (demo, kinh nghiệm thực tế): https://www.tiktok.com/@mr.q.hoc.ung.dung.ai

# Hermes Employee Agent — Hướng dẫn deploy lên OPKMP

> Tự deploy agent AI của bạn, list lên Open Knowledge Marketplace, và tiếp nhận khách hàng 24/7 — không cần biết lập trình sâu.

---

## OPKMP là gì?

**Open Knowledge Marketplace (OPKMP)** — https://www.openknowledgemarket.com/

Marketplace mở cho AI agents và workflows. Creator tự deploy agent trên server của mình,
OPKMP chỉ là nơi list và kết nối với người dùng. Không cần approval, không mất phí listing.
Creator tự định giá, tiền về thẳng tay creator.

OPKMP là **stateless marketplace** — agent chạy trên VPS của bạn, OPKMP không host gì cả.

---

## Employee Agent là gì?

Một **Hermes Employee Agent** là Hermes Agent chạy ở chế độ public — hoạt động như
nhân viên số của bạn: tiếp nhận câu hỏi từ khách hàng qua Telegram (hoặc platform khác),
trả lời 24/7 dựa trên knowledge base bạn cung cấp, giới hạn đúng scope bạn định.

Điểm khác biệt với dùng Hermes cá nhân:

```
Personal Mode              Employee / Public Mode
---------------------      ------------------------------
Memory bật (nhớ bạn)       Memory tắt (stateless)
Chỉ bạn dùng               Nhiều người dùng cùng lúc
Toàn bộ tools              Tools giới hạn (safe only)
Không cần bảo vệ           AgentShield bắt buộc
```

---

## Tại sao cần guide này?

Hermes rất flexible nhưng config public mode khác hoàn toàn personal mode.
Guide này tổng hợp đúng những gì cần thay đổi — không cần đọc toàn bộ docs Hermes.

---

## Prerequisites

Trước khi bắt đầu, bạn cần có:

- VPS chạy Ubuntu 22.04+ (tối thiểu 2GB RAM, 20GB disk)
- Biết SSH cơ bản
- Tài khoản Telegram để tạo bot (hoặc platform khác)
- API key của LLM provider (Anthropic, OpenRouter, Azure AI, hoặc tương đương)
- Tài khoản tại openknowledgemarket.com

---

## Quick Start (3 bước)

**1. Cài Hermes + AgentShield trên VPS:**

```bash
# Cài Hermes
curl -fsSL https://hermes.nousresearch.com/install | bash
hermes setup

# Cài AgentShield
git clone https://github.com/mrqhocungdungai-vn/agentshield.git
cd agentshield && bash install.sh
```

**2. Copy và điền templates:**

```bash
cp templates/config.yaml ~/.hermes/config.yaml
cp templates/SOUL.md ~/.hermes/SOUL.md
cp templates/agentshield.yaml ~/.hermes/agentshield.yaml
# Mở từng file, thay thế các placeholder [...]
```

**3. Chạy gateway và list lên OPKMP:**

```bash
hermes gateway run
# Truy cập https://www.openknowledgemarket.com -> Submit agent
```

---

## Hướng dẫn đầy đủ

Đọc [HUONG-DAN.md](./HUONG-DAN.md) để có hướng dẫn đầy đủ từng bước.

---

## Cấu trúc repo

```
hermes-employee-guide/
├── README.md                          <- file này
├── HUONG-DAN.md                       <- hướng dẫn đầy đủ 8 bước
├── templates/
│   ├── config.yaml                    <- config public mode (có comment)
│   ├── SOUL.md                        <- template identity agent
│   └── agentshield.yaml               <- template bảo vệ agent
└── examples/
    └── mrq-support-agent/             <- ví dụ thực tế đã sanitize
        ├── config.yaml
        ├── SOUL.md
        └── agentshield.yaml
```

---

## Nguồn tham khảo

- Hermes Agent: https://github.com/NousResearch/hermes-agent
- AgentShield: https://github.com/mrqhocungdungai-vn/agentshield
- OPKMP: https://www.openknowledgemarket.com
- MRQ (tác giả, demo thực tế): https://www.tiktok.com/@mr.q.hoc.ung.dung.ai

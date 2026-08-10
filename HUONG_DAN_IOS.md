# AnswerMate iOS — bản test (fork Signal-iOS)

Rebrand của [Signal-iOS](https://github.com/signalapp/signal-ios) (AGPL-3.0) → **AnswerMate**
cho mục đích thử nghiệm trên iPhone.

## Đã đổi gì trong bản này (rebrand)

| Hạng mục | Giá trị |
|---|---|
| Tên hiển thị (home screen) | `AnswerMate` (CFBundleDisplayName) |
| Bundle ID app | `com.miso.answermate` (prefix `SIGNAL_BUNDLEID_PREFIX = com.miso.answermate`) |
| Bundle ID Notification Service (NSE) | `com.miso.answermate.nse` |
| Bundle ID Share Extension | `com.miso.answermate.shareextension` |
| App Groups | `group.com.miso.answermate.signal.group` (+ `.staging`) — code + entitlements đều theo biến nên khớp nhau |
| Icon mặc định | logo AnswerMate (bong bóng chat trắng + chữ A xanh) — `Signal/AppIcons/AppIcon.icon/Assets/logo.svg` |

**CHƯA port tính năng AnswerMate Android** — ĐANG LÀM prototype auto-answer (xem dưới).
Signal-iOS là codebase Swift riêng — các tính năng phải viết lại bằng Swift (dự án
nhiều phiên). Bản test hiện tại để **kiểm tra app cài được + chạy được + giao diện** trên iPhone.

## Prototype auto-answer (M0 — commit d960558d5c)

Code nằm ở `Signal/Calls/CallService.swift` (cuối file): `ForkConfig` (đọc
Documents/fork_config.txt — sửa qua app Files, hot reload) + `ForkAutoAnswer`
(quan sát `CallServiceState` → khi có call đến + RingRTC ready → gọi
`individualCallService.handleAcceptCall` — đúng đường user bấm nút).

Config keys:
- `auto_answer_enabled=true` — bật/tắt
- `auto_answer_delay_ms=1500` — delay trước khi tự trả lời
- `auto_answer_allowlist=` — số E.164 phân cách phẩy (bỏ dấu +); trống = tất cả

⚠️ Giới hạn iOS cần test trên máy thật: khi khoá màn hình/background, iOS có thể
không giữ audio session; CallKit vẫn hiện màn hình cuộc gọi đến rồi tự chuyển
connected. Cellular (gọi số thường) KHÔNG auto-answer được (CallKit không cho app
tự trả lời — chỉ Signal-to-Signal call là làm được).

## Build — chạy trên GitHub Actions macOS runner (MIỄN PHÍ)

Máy Linux KHÔNG build được iOS (bắt buộc macOS + Xcode). Pipeline này build trên runner
`macos-26` của GitHub (Xcode 26.6 — đúng bản Signal yêu cầu). **Repo PUBLIC → macOS
runner miễn phí không giới hạn** (standard runners).

1. Tạo repo public + push (1 lần):
   ```bash
   gh repo create <ten-repo> --public --source . --push
   ```
2. Vào tab **Actions** → workflow **Build AnswerMate iOS (test)** → **Run workflow** (hoặc
   tự chạy mỗi lần push main).
3. Chờ build (lần đầu 30–60 phút: tải deps + compile). Xong → mở run → mục **Artifacts**
   → tải `AnswerMate-iOS-test-unsigned.ipa`.

## Cài lên iPhone (sideload — không cần trả $99)

File .ipa KHÔNG ký (Code Signing Off). Cách cài:

- **Cần 1 máy Windows hoặc macOS** (không làm từ Linux được):
  - [AltStore](https://altstore.io): cài companion app trên máy tính → cài AltStore vào iPhone
    → thả file .ipa vào AltStore (hoặc mở file bằng AltStore trên iPhone).
  - Hoặc [Sideloadly](https://sideloadly.io) (Windows/macOS): chọn .ipa + Apple ID → Install.
- Apple ID **miễn phí** là đủ. Hạn chế của tài khoản free:
  - App hết hạn sau **7 ngày** → cần refresh (AltStore làm được, phải nối lại máy tính).
  - **Push notification (APNs) KHÔNG hoạt động** → không đăng ký được tài khoản Signal mới,
    không nhận/gửi tin được. Test build chỉ để xem app cài + chạy + UI + không crash.
- Muốn test đầy đủ (đăng ký, nhắn tin, gọi) → cần Apple Developer Program ($99/năm) + TestFlight.

## Cấu trúc repo

- `Signal.xcodeproj` — project (đổi bundle id trong `project.pbxproj`)
- `Signal/AppIcons/AppIcon.icon/Assets/logo.svg` — icon mặc định (định dạng icon-composition Xcode 26)
- `.github/workflows/build-ios.yml` — pipeline build test
- Rebrand cũng cần sửa (nếu port tiếp): `SignalServiceKit/Environment/TSConstants.swift`
  (applicationGroup — tự theo `bundleIdPrefix`), `Bundle+OWS.swift` (fallback đã đổi).

## Lưu ý bản quyền

AGPL-3.0: fork dùng riêng OK; nếu phân phối phải mở source. Không dùng logo/tên "Signal".
Icon AnswerMate tự vẽ (bong bóng chat + chữ A) — không dùng logo Signal.

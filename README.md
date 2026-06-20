# Larvoice TTS API

Larvoice TTS API giúp tạo giọng nói từ văn bản qua HTTP API. API chạy bất đồng bộ: bạn gửi text để tạo job, sau đó poll trạng thái job cho đến khi có file audio.

Base URL:

```text
https://api.larvoice.com
```

## Mục Lục

- [Xác thực](#xác-thực)
- [Endpoint](#endpoint)
- [Playground](#playground)
- [Quick start](#quick-start)
- [Thông tin tài khoản](#thông-tin-tài-khoản)
- [Tạo job TTS](#tạo-job-tts)
- [Kiểm tra trạng thái job](#kiểm-tra-trạng-thái-job)
- [Danh sách job](#danh-sách-job)
- [File output](#file-output)
- [Tham số request](#tham-số-request)
- [Upload voice mẫu](#upload-voice-mẫu)
- [Rate limit và quota](#rate-limit-và-quota)
- [Lỗi thường gặp](#lỗi-thường-gặp)
- [Chính sách dữ liệu](#chính-sách-dữ-liệu)

## Xác Thực

Gửi API key trong header `x-api-key`:

```http
x-api-key: <your-api-key>
```

API key chỉ xem được job do chính key đó tạo. Nếu mất key, hãy yêu cầu cấp key mới.

## Endpoint

```http
GET  /health
GET  /playground
GET  /v1/me
POST /v1/voices
POST /v1/tts
GET  /v1/tts/jobs
GET  /v1/tts/jobs/:job_id
GET  /files/:file?expires=...&signature=...
HEAD /files/:file?expires=...&signature=...
```

## Playground

Trải nghiệm API trực tiếp tại:

```text
https://api.larvoice.com/playground
```

Playground cho phép nhập API key, upload voice mẫu, kiểm tra quota, tạo job TTS, poll trạng thái và nghe audio sau khi job hoàn thành.

## Quick Start

### 1. Kiểm tra service

```bash
curl --location 'https://api.larvoice.com/health'
```

Response:

```json
{"ok":true}
```

### 2. Tạo job TTS

```bash
curl --location 'https://api.larvoice.com/v1/tts' \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: <your-api-key>' \
  --data '{
    "ref_audio_url": "https://example.com/voice-sample.mp3",
    "language": "vi",
    "gen_text": "Xin chào, đây là Larvoice TTS API.",
    "return_srt": true,
    "output_format": "wav"
  }'
```

Response `202 Accepted`:

```json
{
  "data": {
    "job_id": "9f6c2131-0f7c-41e4-90f2-ae3c7f336d00",
    "status": "queued",
    "character_count": 34,
    "status_url": "https://api.larvoice.com/v1/tts/jobs/9f6c2131-0f7c-41e4-90f2-ae3c7f336d00"
  }
}
```

### 3. Poll job

```bash
curl --location 'https://api.larvoice.com/v1/tts/jobs/9f6c2131-0f7c-41e4-90f2-ae3c7f336d00' \
  --header 'x-api-key: <your-api-key>'
```

Khi job hoàn thành:

```json
{
  "data": {
    "job_id": "9f6c2131-0f7c-41e4-90f2-ae3c7f336d00",
    "status": "completed",
    "character_count": 34,
    "created_at": "2026-06-20T12:00:00.000Z",
    "updated_at": "2026-06-20T12:00:10.000Z",
    "started_at": "2026-06-20T12:00:05.000Z",
    "completed_at": "2026-06-20T12:00:10.000Z",
    "audio_url": "https://api.larvoice.com/files/tts_20260620_120010_123456.wav?expires=1782043210&signature=...",
    "content_type": "audio/wav",
    "duration_seconds": 2.18,
    "expires_at": "2026-06-21T12:00:10.000000",
    "return_srt": true,
    "srt_url": "https://api.larvoice.com/files/tts_20260620_120010_123456_aligned.srt?expires=1782043210&signature=...",
    "segments": []
  }
}
```

## Thông Tin Tài Khoản

Endpoint:

```http
GET /v1/me
```

Dùng endpoint này để kiểm tra quota, số ký tự đã dùng, số ký tự còn lại và hạn dùng API key.

Ví dụ:

```bash
curl --location 'https://api.larvoice.com/v1/me' \
  --header 'x-api-key: <your-api-key>'
```

Response:

```json
{
  "data": {
    "id": "6fd50e12-52ae-4b83-a9b1-0848f64cbe15",
    "name": "customer-a",
    "key_prefix": "lv_xxxxxxxx",
    "character_limit": 100000,
    "characters_used": 3400,
    "remaining_characters": 96600,
    "status": "active",
    "created_at": "2026-06-20T12:00:00.000Z",
    "last_used_at": "2026-06-20T12:10:00.000Z",
    "expires_at": null
  }
}
```

`expires_at: null` nghĩa là API key chưa có ngày hết hạn cố định.

## Tạo Job TTS

Endpoint:

```http
POST /v1/tts
```

Headers:

```http
Content-Type: application/json
x-api-key: <your-api-key>
```

Body tối thiểu:

```json
{
  "ref_audio_url": "https://example.com/voice-sample.mp3",
  "language": "vi",
  "gen_text": "Nội dung cần tạo giọng nói."
}
```

Body có SRT:

```json
{
  "ref_audio_url": "https://example.com/voice-sample.mp3",
  "language": "vi",
  "gen_text": "Nội dung cần tạo giọng nói.",
  "return_srt": true,
  "output_format": "wav"
}
```

## Kiểm Tra Trạng Thái Job

Endpoint:

```http
GET /v1/tts/jobs/:job_id
```

Status có thể là:

| Status | Ý nghĩa |
| --- | --- |
| `queued` | Job đang chờ xử lý. |
| `processing` | Job đang được tạo audio. |
| `completed` | Job hoàn thành, response có `audio_url`. |
| `failed` | Job thất bại, response có `error_code` và `error_message`. |

Ví dụ:

```bash
curl --location 'https://api.larvoice.com/v1/tts/jobs/<job_id>' \
  --header 'x-api-key: <your-api-key>'
```

## Danh Sách Job

Endpoint:

```http
GET /v1/tts/jobs
```

Ví dụ:

```bash
curl --location 'https://api.larvoice.com/v1/tts/jobs?limit=20' \
  --header 'x-api-key: <your-api-key>'
```

Query params:

| Param | Mô tả |
| --- | --- |
| `limit` | Số job mỗi trang. Mặc định `20`, tối đa `100`. |
| `cursor` | Cursor trang kế tiếp từ response trước. |
| `status` | Lọc theo `queued`, `processing`, `completed`, `failed`. |

Response có `pagination.next_cursor` nếu còn trang tiếp theo.

## File Output

Khi job `completed`, dùng `audio_url` để tải file audio. Nếu request có `return_srt: true`, response có thêm `srt_url`.

Output URL có dạng:

```text
https://api.larvoice.com/files/<file>?expires=<unix>&signature=<hmac>
```

Lưu ý:

- Link tải có thời hạn, tối đa `24` giờ.
- Không chia sẻ public link nếu audio chứa nội dung riêng tư.
- Có thể dùng `HEAD` để kiểm tra metadata file trước khi tải.
- Nếu link hết hạn, API trả `403 Forbidden`.

## Tham Số Request

| Field | Bắt buộc | Kiểu | Mặc định | Mô tả |
| --- | --- | --- | --- | --- |
| `post_speed` | Không | number | `1.1` | Tốc độ hậu xử lý, từ `0.5` đến `2`. |
| `post_pitch` | Không | number | `0` | Chỉnh cao độ, từ `-6` đến `6`. |
| `post_volume` | Không | number | `0.1` | Tăng/giảm âm lượng, từ `-20` đến `20`. |
| `language` | Không | string | `vi` | `vi` hoặc `en`. |
| `gen_text` | Có | string | - | Nội dung cần tạo giọng nói. Tối đa `10000` ký tự mỗi job. |
| `ref_audio_url` | Có | string | - | URL giọng mẫu HTTPS bất kỳ hoặc URL trả về từ `POST /v1/voices`. Larvoice cache sang R2 và dùng tối đa `2.5` giây đầu làm reference. |
| `return_srt` | Không | boolean | `false` | Trả thêm file SRT và segments. |
| `output_format` | Không | string | `wav` | `wav` hoặc `mp3`. |

Các field lạ sẽ bị reject. `output_audio=true` không được hỗ trợ trong queued API; hãy dùng `audio_url` sau khi job hoàn thành.

## Upload Voice Mẫu

Endpoint:

```http
POST /v1/voices
```

Upload file audio để Larvoice cache sang R2 và trả về `ref_audio_url` chuẩn dùng cho `POST /v1/tts`.

Ví dụ:

```bash
curl --location 'https://api.larvoice.com/v1/voices' \
  --header 'x-api-key: <your-api-key>' \
  --form 'file=@voice-sample.mp3'
```

Response:

```json
{
  "data": {
    "ref_audio_url": "https://api.larvoice.com/ref-audio/<sha256>.mp3",
    "file_name": "<sha256>.mp3",
    "content_type": "audio/mpeg",
    "size_bytes": 123456,
    "max_reference_seconds": 2.5
  }
}
```

## Rate Limit Và Quota

- Mỗi API key có quota ký tự riêng.
- Mỗi request TTS trừ quota theo số ký tự trong `gen_text`.
- Nếu job thất bại sau retry, quota được hoàn lại.
- Rate limit mặc định: `50` request/phút/API key.

## Lỗi Thường Gặp

Error response chuẩn:

```json
{
  "error": {
    "code": "validation_error",
    "message": "Invalid request",
    "details": [
      {
        "field": "gen_text",
        "message": "Parameter must be a non-empty string"
      }
    ]
  },
  "request_id": "01J..."
}
```

| HTTP | Code | Ý nghĩa |
| --- | --- | --- |
| `400` | `validation_error` | Body sai hoặc thiếu field. |
| `401` | `unauthorized` | Thiếu hoặc sai API key. |
| `402` | `quota_exceeded` | Không đủ quota ký tự. |
| `403` | `forbidden` | API key không được phép gọi endpoint này. |
| `404` | `not_found` | Không tìm thấy job hoặc file. |
| `413` | `payload_too_large` | Body JSON quá lớn. |
| `429` | `rate_limited` | Vượt rate limit. |
| `502` | `tts_failed` | Không tạo được audio. |
| `504` | `tts_timeout` | Xử lý quá thời gian. |

## Chính Sách Dữ Liệu

Larvoice TTS API được thiết kế theo nguyên tắc lưu tối thiểu.

- Không lưu trữ dài hạn nội dung text khách hàng gửi lên.
- Dữ liệu request, trạng thái job, audio output và SRT output chỉ tồn tại tối đa `24` giờ theo chính sách vận hành.
- Sau `24` giờ, link tải hết hạn và dữ liệu job/output được đưa vào luồng dọn dẹp định kỳ.
- Output chỉ truy cập được bằng signed URL có `expires` và `signature`; không có public file listing.
- API không trả raw response nội bộ, path máy chủ, model nội bộ, log kỹ thuật hoặc domain nội bộ.
- Log public tối thiểu, ưu tiên `request_id`, mã lỗi chuẩn, thời gian xử lý và trạng thái; không dùng log để lưu raw text/audio.
- API key chỉ hiển thị một lần khi được cấp; hệ thống chỉ lưu hash, prefix, quota và trạng thái key.
- Dữ liệu khách hàng không được bán, chia sẻ cho bên thứ ba không liên quan, hoặc dùng để huấn luyện model.
- Khi cần xóa dữ liệu sớm, liên hệ Larvoice kèm `job_id` hoặc API key liên quan.

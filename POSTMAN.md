# Larvoice Postman

Import Larvoice API vào Postman bằng OpenAPI.

## Import bằng URL

1. Mở Postman.
2. Chọn `Import`.
3. Chọn `Link`.
4. Dán URL:

```text
https://api.larvoice.com/openapi.json
```

5. Bấm `Import`.

## Variables

Sau khi import, tạo environment riêng cho Larvoice:

1. Chọn `Environments` ở sidebar trái.
2. Bấm `+` để tạo environment mới.
3. Đặt tên, ví dụ `Larvoice API`.
4. Thêm các variables bên dưới.
5. Bấm `Save`.
6. Ở góc trên bên phải Postman, chọn environment `Larvoice API` trước khi chạy request.

Variables cần thêm:

| Variable | Value |
| --- | --- |
| `base_url` | `https://api.larvoice.com` |
| `api_key` | API key Larvoice của bạn |

Larvoice dùng header:

```http
x-api-key: {{api_key}}
```

Nếu collection import chưa tự dùng `{{base_url}}`, mở collection settings và đặt base URL thành:

```text
{{base_url}}
```

## Test nhanh

Chạy các request theo thứ tự:

1. `GET /health`
2. `GET /v1/me`
3. `GET /v1/languages`
4. `POST /v1/voices`
5. `POST /v1/tts/preview`
6. `POST /v1/tts`
7. `GET /v1/tts/jobs/{job_id}`

## Upload voice

Request:

```http
POST /v1/voices
```

Body type: `form-data`

| Key | Type | Required | Note |
| --- | --- | --- | --- |
| `file` | File | Yes | `mp3`, `wav`, `m4a`, tối đa `25 MB` |
| `name` | Text | No | Tên voice |
| `ref_text` | Text | No | Nội dung nghe được trong voice mẫu |

Sau khi upload, copy `voice_id` để dùng cho preview hoặc tạo job TTS.

## Tạo preview

Request:

```http
POST /v1/tts/preview
```

Body JSON mẫu:

```json
{
  "voice_id": "{{voice_id}}",
  "text": "Xin chào, đây là bản nghe thử.",
  "language": "vi",
  "output_format": "wav"
}
```

## Tạo job TTS

Request:

```http
POST /v1/tts
```

Body JSON mẫu:

```json
{
  "voice_id": "{{voice_id}}",
  "gen_text": "Nội dung cần tạo giọng nói.",
  "language": "vi",
  "output_format": "wav",
  "return_srt": false
}
```

Response trả `job_id`. Dùng `GET /v1/tts/jobs/{job_id}` để kiểm tra trạng thái.

## Lỗi thường gặp

| Status | Code | Cách xử lý |
| --- | --- | --- |
| `401` | `unauthorized` | Kiểm tra `api_key` trong environment. |
| `400` | `validation_error` | Kiểm tra body JSON hoặc form-data. |
| `403` | `voice_limit_exceeded` | Xóa voice cũ hoặc nâng giới hạn. |
| `413` | `payload_too_large` | Giảm kích thước file upload. |
| `429` | `rate_limited` | Chờ rồi thử lại. |

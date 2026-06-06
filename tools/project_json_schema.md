# project.json スキーマ

## 目的

ChatGPTが作成した動画素材を、AutoMovieBuilderが迷わず読み込めるようにする。

## 必須項目

| キー | 型 | 内容 |
|---|---|---|
| `video_type` | string | `short` または `longform` |
| `video_mode` | string | `short_vertical` または `long_horizontal` |
| `title` | string | YouTube投稿タイトル |
| `recommended_publish_at` | string | 推奨公開日時。形式は `YYYY-MM-DD HH:MM` |
| `images_zip` | string | 画像ZIPファイル名。通常は `images.zip` |
| `srt` | string | VOICEVOX用SRTファイル名。通常は `voicevox.srt` |
| `description` | string | 説明文ファイル名。通常は `description.txt` |
| `fixed_comment` | string | 固定コメントファイル名。通常は `fixed_comment.txt` |
| `status` | string | `ready` のものだけ自動生成対象 |

## 任意項目

| キー | 型 | 内容 |
|---|---|---|
| `chapters` | string | 長尺用チャプター。通常は `chapters.txt` |
| `priority` | number | 作成優先度 |
| `memo` | string | 制作メモ |
| `source` | string | テーマ選定の根拠 |

## ショート動画サンプル

```json
{
  "video_type": "short",
  "video_mode": "short_vertical",
  "title": "美容室の予約管理、まだ紙でやってる？",
  "recommended_publish_at": "2026-06-10 21:00",
  "images_zip": "images.zip",
  "srt": "voicevox.srt",
  "description": "description.txt",
  "fixed_comment": "fixed_comment.txt",
  "status": "ready"
}
```

## 長尺動画サンプル

```json
{
  "video_type": "longform",
  "video_mode": "long_horizontal",
  "title": "小さなお店の売上・在庫管理、Excelでここまでできます【初心者向け】",
  "recommended_publish_at": "2026-06-15 20:00",
  "images_zip": "images.zip",
  "srt": "voicevox.srt",
  "description": "description.txt",
  "fixed_comment": "fixed_comment.txt",
  "chapters": "chapters.txt",
  "status": "ready"
}
```

## バリデーションルール

- `status` が `ready` 以外なら自動生成対象外。
- `video_mode` が `short_vertical` なら1080x1920で生成。
- `video_mode` が `long_horizontal` なら1920x1080で生成。
- `recommended_publish_at` が空の場合は、設定値の標準公開時間を使う。
- `images_zip` と `srt` は必須。
- 画像数とSRTブロック数が一致しない場合は動画生成しない。

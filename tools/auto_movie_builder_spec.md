# Auto Movie Builder 仕様書

## 目的

GitHubに保存されたYouTube動画素材から、画像ZIPとVOICEVOX用SRTを使って、クリックだけでMP4動画と投稿用テキスト一式を自動生成する。

## 前提

- 現行ツールは `app.py` の `build_video()` を中核として使う。
- 設定はユーザー配下の `.short_video_maker/settings.json` に保存する。
- VOICEVOX Engineは起動済み前提。
- Pythonは3.10〜3.12を推奨。
- MoviePyは1.0.3固定。
- FFmpegが必要。

## 作るEXE

### 1. MovieMakerSettings.exe

設定専用UI。

保存する設定：

- GitHub素材ルートフォルダ
- 出力先フォルダ
- BGMフォルダ
- BGM音量
- 音声音量
- VOICEVOX話者ID
- 話者プリセット
- 読み上げ速度
- 抑揚
- 音声後の余裕秒
- トランジション
- 切替秒
- フェードイン
- フェードアウト
- FPS
- 画像表示方法 cover / contain
- 軽いズーム有無
- ショート標準公開時間
- 長尺標準公開時間
- 作成済みスキップ
- 強制再作成
- エラー時に次の動画へ進む

### 2. AutoMovieBuilder.exe

自動動画作成専用。

処理内容：

1. settings.jsonを読む。
2. GitHub素材ルートを探索する。
3. `project.json` を持つ動画プロジェクトを探す。
4. `status` が `ready` のものだけ対象にする。
5. `.build_done.json` があるものはスキップする。ただし強制再作成ONなら作る。
6. `images.zip` を展開する。
7. `voicevox.srt` を読む。
8. `video_mode` に応じてショート/長尺を自動切替する。
9. `build_video()` でMP4を生成する。
10. output配下に公開推奨日時フォルダを作る。
11. 動画、説明文、固定コメント、タイトル、チャプターをコピーする。
12. `source_project.json` と `build_log.txt` を出力する。
13. 元プロジェクトに `.build_done.json` を作る。

## GitHub素材フォルダ構成

```text
assets/
  shorts/
    2026-06-week2/
      01_動画名/
        images.zip
        voicevox.srt
        description.txt
        fixed_comment.txt
        title.txt
        project.json

  longform/
    2026-06-week2/
      01_動画名/
        images.zip
        voicevox.srt
        description.txt
        fixed_comment.txt
        title.txt
        chapters.txt
        project.json
```

## project.json仕様

### ショート

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

### 長尺

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

## outputフォルダ構成

```text
output/
  2026-06-10_21-00_美容室の予約管理/
    video.mp4
    title.txt
    description.txt
    fixed_comment.txt
    source_project.json
    build_log.txt

  2026-06-15_20-00_小さなお店の売上在庫管理/
    video.mp4
    title.txt
    description.txt
    fixed_comment.txt
    chapters.txt
    source_project.json
    build_log.txt
```

## 作成済み判定

動画作成後、元プロジェクトフォルダに `.build_done.json` を作成する。

```json
{
  "built_at": "2026-06-07 10:30:00",
  "output_path": "output/2026-06-10_21-00_美容室の予約管理/video.mp4",
  "status": "done"
}
```

## 必要な追加関数

| 関数 | 内容 |
|---|---|
| `scan_video_projects(root)` | GitHub素材ルートから `project.json` を探す |
| `load_project_meta(path)` | `project.json` を読む |
| `extract_images_zip(project_dir, zip_name)` | `images.zip` を展開する |
| `build_project(project_dir, meta, settings)` | 1動画を作る |
| `build_all_ready_projects()` | 未作成動画を一括生成する |
| `copy_publish_files()` | 投稿用テキストをoutputにコピーする |
| `write_build_done()` | 作成済みフラグを書く |
| `sanitize_folder_name()` | 公開日時 + タイトルの安全なフォルダ名を作る |

## エラーハンドリング

- VOICEVOX未起動なら処理を止める。
- `images.zip` がない場合はエラー。
- `voicevox.srt` がない場合はエラー。
- 画像数とSRTブロック数が一致しない場合はエラー。
- 1本で失敗しても、設定で「エラー時に次へ進む」がONなら次の動画へ進む。
- すべての結果を `build_log.txt` に保存する。

## 実装方針

最初は現行 `app.py` を壊さず、以下の追加ファイルで実装する。

```text
settings_ui.py
auto_builder.py
```

共通処理は、後で `movie_core.py` に分離する。

## EXE化想定

```text
MovieMakerSettings.exe
AutoMovieBuilder.exe
```

PyInstallerでそれぞれ個別にexe化する。

# 牧養材料 Dump

從 `https://www.churchintaichung.org/shepherding_materials/seriesxx/Axx_xx.pdf` dump PDF，轉成純文字，並提供一個可在手機與電腦閱讀的網頁(頁面設計與TWGBR-DRMs相似)。

感謝台中市召會提供原始檔案。

## 專案結構

| 路徑 | 說明 |
| --- | --- |
| `data/` | 轉換完成的文字檔 `AXX_XXs.txt`（系列 01 至 06，每系列 16 篇），為最終成品 |
| `config.json` | 篇目清單與本週進度設定，供 `index.html` 讀取 |
| `index.html` | 閱讀網頁，每次開啟直接顯示本週進度 |
| `dumpAll.py` | 批次下載、轉檔全部系列，並重新產生完整的 `config.json` |
| `title.py` | 設定本週進度，同步更新 `config.json` 與 `index.html` |
| `autoDump.sh` | 下載單篇的 shell 腳本 |
| `heritage/` | 舊版程式與網頁（`shepher.py`、`autoDump-x.sh`、`index_v2.html`、`autodump.tar.gz`），僅供留存 |

## 環境需求

類UNIX/POSIX系統CLI。

需先安裝：

- Python 3.10 以上
- `curl`
- `pdftohtml`（poppler 套件）
- `w3m`

## 使用方式

### 下載單篇

```sh
curl -O "https://raw.githubusercontent.com/user-7510/shepherding_materials_dump/refs/heads/main/autoDump.sh"
chmod +x autoDump.sh
./autoDump.sh
```

成品為 `AXX_XXs.txt`。

### 批次下載全部系列

```sh
python3 dumpAll.py
python3 dumpAll.py --series-count 6 --chapters-per-series 16
python3 dumpAll.py --delay 1.0
python3 dumpAll.py --push
```

- 文字檔輸出至 `./data`，並產生全新的 `config.json`。
- 來源網站若缺少某一篇，該篇會被跳過並印出警告，不會中斷整個批次。
- `--delay` 為每篇下載間隔秒數，預設 0.5。
- `--push` 於完成後自動執行 `git add`、`commit`、`push`。
- 重新產生 `config.json` 會清除本週主題文字，執行後請再跑一次 `title.py`。

### 設定本週進度

```sh
python3 title.py
python3 title.py --series 02 --chapter 07 --topic 生命的話
python3 title.py --order desc
```

- 不帶參數時會以互動方式詢問系列、章節與主題文字。
- 該篇必須已存在於 `config.json`，本程式只切換預設，不會下載新內容。
- 該篇會置頂並成為預設，其餘篇目依系列、章節編號排序：預設由小到大，`--order desc` 改為由大到小。
- 主題文字寫入 `config.json` 的 `weeklyTopic`，留白則移除。
- 程式不會自動推送，修改後請自行 `git commit` 並 `git push`，網頁才會更新。

## 閱讀網頁

`index.html` 會從本儲存庫的 `config.json` 讀取資料。

- 每次開啟一律載入本週進度，不沿用上次選擇的篇目。
- 標題列的「本週進度」按鈕可隨時跳回本週篇目。
- 底部「切換篇目」可閱讀其他篇目。
- 「⋮」選單提供字體縮放與深色模式。
- 設定檔與文字檔的請求皆避開快取，推送後通常很快就能看到更新。

## `config.json` 格式

```json
{
    "textFileUrl": "本週進度文字檔的網址",
    "weeklyTopic": "本週主題文字（可省略）",
    "chapters": {
        "系列六 第十二題": "文字檔網址"
    }
}
```

`textFileUrl` 對應的篇目即為本週進度，且位於 `chapters` 的第一項。

## 更新流程

1. 如有新篇目，執行 `python3 dumpAll.py`。
2. 執行 `python3 title.py` 設定本週進度。
3. 推送至 GitHub：

```sh
git add -A
git commit -m "更新本週進度"
git push
```

## 致謝

- 感謝台中市召會提供原始檔案。
- `shepher.py`（現存於 `heritage/`）由 H.L.2026 撰寫。

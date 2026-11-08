# 风格图谱 · 数据说明

网站分成「页面」和「数据文件」两部分。页面 `index.html` 只放界面和代码，打开时用 `<script>` 标签加载下面这些文件。

**为什么是 .js 而不是 .json / .md**：在 claude.ai 的查看器里，页面用 fetch 读不到同目录的文件（会报 Load failed），但可以用 `<script>` 加载。所以每个数据文件都是一段给全局变量赋值的脚本，内容就是 JSON。文件一律用纯 ASCII 写（`json.dumps(..., ensure_ascii=True)`），这样不受文件编码影响。

更新内容时只改对应的数据文件，并且只发布改动过的文件；没有改的文件会原样保留。

## 文件结构

| 路径 | 内容 |
| --- | --- |
| `index.html` | 页面和代码。只有改界面或新增功能时才动它 |
| `data/catalog.js` | 风格 `styles`、待学风格 `nextStyles`、手册目录 `manuals`、技巧 `techniques`、学习路线 `path`、参考听单 `listen`、速查 `cheats`、术语 `glossary`，以及更新日期 `updated` |
| `data/manuals/<id>.js` | 每份手册的 Markdown 全文，写成 `(window.ATLAS_MD=window.ATLAS_MD\|\|{})["<id>"]="…";`，文件名和 `manuals[].id` 一致 |
| `data/songs.js` | 曲目库 `songs`，以及可试听、可下载的音频清单 `audio`（以曲目 id 为键） |
| `data/stuck.js` | 卡点与对策 `entries` |
| `audio/<曲目 id>/` | 音频文件，供页面播放。单个文件不超过 15 MB，整首用 MP3 320 kbps |
| `audio/<曲目 id>/<文件>.b64.js` | 同一音频的 base64 副本，下载时用（查看器里读不到二进制文件）。写成 `(window.ATLAS_BIN=window.ATLAS_BIN\|\|{})["<音频路径>"]="…";`。base64 会大约多出三分之一，整首 MP3 约 12.8 MB，仍在 15 MB 以内 |

## 常见更新

**加一份手册**
1. 在 `data/manuals/` 下新建 `<id>.js`（Docs 手册导出为 Markdown，再按上面的格式写成脚本）。
2. 在 `catalog.js` 的 `manuals` 里加一条：`id`、`style`、`cat`、`date`、`doc`（原 Docs 手册的 id，没有就写 null）、`title`、`sub`、`meta`、`devices`、`sum`、`keys`、`tech`、`secs`。
3. `tech` 里如果用到新技巧，先在 `techniques` 里加上（`id`、`name`、`kind`、`line`、`how`）。技巧出现在哪些手册里，页面会自动算出来。
4. 合适的话，把它放进 `path` 的某一阶段。

**加一首曲目**：在 `songs.js` 的 `songs` 里加一条（`id`、`t`、`a`、`al`、`y`、`style`、`role`、`src`、`feel`、`hear`、`harm`、`facts`、`kw`、`tech`）。`src` 写出处手册的 id，没有就写空数组。

**给曲目加音频**：把文件放到 `audio/<曲目 id>/`，同时生成它的 `.b64.js` 副本，再在 `songs.js` 的 `audio` 里以曲目 id 为键登记 `song` 和 `files`（`path`、`name`、`label`、`note`、`mb`、`play`）。下载时会打包成 zip。

**加一个卡点**：在 `stuck.js` 的 `entries` 里加一条。必填：`id`、`date`、`status`（open / doing / done）、`kinds`、`title`、`where`、`stuck`、`tried`、`triedCost`、`diag`、`now`、`future`、`manuals`、`tech`。可选：`checks`、`explain`、`prep`、`plan`、`vocab`、`ref`、`arr`。

**加一个新风格**：在 `catalog.js` 的 `styles` 里加一条（`id`、`name`、`status`、`gist`、`traits`），并把它从 `nextStyles` 里移出去。

## 约定

- 改完把 `catalog.js` 的 `updated` 改成当天日期。
- id 只用小写字母、数字和连字符，一旦用了就不要改，进度记录靠它保存。
- 发布前先列出线上文件（Artifact 的 list 加 `scope: "files"`），以线上版本为准，避免覆盖别的会话的改动。
- 音频版权属于原作者，只用于个人学习。网站设为公开前，先把 `audio/` 拿掉，或者给网站加访问限制。
- `catalog.js`、`songs.js`、`stuck.js` 开头都是 `window.ATLAS=window.ATLAS||{};` 再赋值，改的时候保持这个形状。

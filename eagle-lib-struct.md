# Eagle 图库目录结构

## 顶层结构

```
{库名}.library/
  metadata.json          ← 文件夹树结构（folders）
  mtime.json             ← 图片修改时间索引，以及 all 总数
  tags.json              ← 标签历史/收藏标签
  actions.json           ← 操作记录（可能为空数组）
  saved-filters.json     ← 保存的筛选器（可能为空数组）
  backup/                ← 备份目录
  images/                ← 所有图片存放目录
    {id}.info/           ← 每张图片一个目录，以 id 命名
    ...
```

## 图片目录（`.info`）

每个 `{id}.info/` 目录内包含：

```
{id}.info/
  metadata.json                          ← 图片元数据
  {name}.{ext}                           ← 原图文件
  {name}_thumbnail.png                   ← 缩略图（大多数存在）
```

**关键结论：文件用 `name` 字段命名，不是 `id`。**

示例（id=`M4QVOIHV55X4K`）：

```
M4QVOIHV55X4K.info/
  metadata.json
  001-xxx世界.jpg
  001-xxx世界_thumbnail.png
```

## metadata.json（图片级）

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `id` | string | 唯一标识，同时是 `.info` 目录名的前缀 | `"M4QVOIHV55X4K"` |
| `name` | string | 显示名称，**实际文件以此命名** | `"001-女全裸的世界"` |
| `ext` | string | 文件扩展名（不含点） | `"jpg"` |
| `size` | number | 文件大小（字节） | `5292557` |
| `width` | number | 图片宽度 | `4000` |
| `height` | number | 图片高度 | `5656` |
| `folders` | string[] | 所属文件夹 ID 列表 | `["M4R0VHEM5FS69"]` |
| `tags` | string[] | 标签列表 | `[]` |
| `url` | string | 来源 URL | `"https://www.pixiv.net/..."` |
| `isDeleted` | boolean | 是否已删除 | `false` |
| `deletedTime` | number | 删除时间戳；仅部分图片存在 | `1772248458716` |
| `btime` | number | 创建时间戳（毫秒） | `1772248458713` |
| `mtime` | number | 修改时间戳（毫秒） | `1772248458716` |
| `lastModified` | number | 原文件最后修改时间戳；通常存在 | `1772248458716` |
| `modificationTime` | number | Eagle 内部修改时间戳 | `1772248458716` |
| `noThumbnail` | boolean | 是否没有缩略图；仅部分图片存在 | `true` |
| `noPreview` | boolean | 是否没有预览；仅部分非普通图片存在 | `true` |
| `star` | number | 星标评分；仅部分图片存在 | `1` |
| `resolutionWidth` | number | 分辨率宽度；仅部分媒体存在 | `1920` |
| `resolutionHeight` | number | 分辨率高度；仅部分媒体存在 | `1080` |
| `duration` | number | 时长；仅部分动态图片/视频类媒体存在 | `10.5` |
| `palettes` | array | 调色板信息 | `[...]` |
| `annotation` | string | 备注/注释 | `""` |

## metadata.json（库级）

库根目录的 `metadata.json` 包含 `folders` 数组，描述文件夹树：

```json
{
  "applicationVersion": "4.x",
  "folders": [
    {
      "id": "文件夹ID",
      "name": "文件夹名称",
      "children": [...]
    }
  ],
  "smartFolders": [],
  "quickAccess": [],
  "tagsGroups": [],
  "modificationTime": 1772248458716
}
```

## 未分类图片

Eagle 的“未分类”不是 `{库名}.library/` 里的真实目录，也不会出现在库级 `metadata.json` 的 `folders` 树中。

判断规则：

- 图片级 `metadata.json` 中 `folders` 是空数组 `[]`，表示该图片不属于任何用户文件夹。
- 如果 `folders` 字段缺失，也应按未分类处理。
- Eagle 正常图库视图会排除 `isDeleted: true` 的图片；因此可见“未分类”应统计 `folders.length === 0 && isDeleted !== true`。
- 需要先遍历 `images/{id}.info/metadata.json`，把 `folders.length === 0` 的图片额外归集成一个虚拟目录，例如“未分类”。

在 `/Users/a1111/Pictures/frog.library` 样本中验证到：

- 库级 `metadata.json` 的 `folders` 为 `[]`。
- `images/` 下有 19 个 `.info` 图片目录。
- 19 个图片级 `metadata.json` 的 `folders` 都是 `[]`。
- 其中 10 个是 `isDeleted: true`，Eagle 可见“未分类”为剩余 9 个未删除图片。

在 `/Users/a1111/Pictures/otherworld.library` 样本中再次验证到：

- 库级 `metadata.json` 有 9 个顶层文件夹，递归后共 115 个真实文件夹 ID。
- `images/` 下有 4636 个 `.info` 图片目录。
- 4577 个图片的 `folders.length === 1`，且引用的文件夹 ID 都能在库级 `folders` 树中找到。
- 59 个图片的 `folders` 为 `[]`，其中 7 个是 `isDeleted: true`。
- Eagle 可见“未分类”为剩余 52 个未删除图片，即 `folders.length === 0 && isDeleted !== true`。
- 没有发现图片引用不存在的文件夹 ID。

示例：

```json
{
  "id": "图片ID",
  "name": "文件名",
  "ext": "jpg",
  "folders": []
}
```

实现上可以使用一个固定的虚拟 ID，例如 `__uncategorized__`，但它不是 Eagle 原生写入 metadata 的文件夹 ID。

## 其他辅助文件

`mtime.json` 是一个以图片 id 为 key 的修改时间索引，并包含 `all` 字段表示图片总数：

```json
{
  "图片ID": 1772248458716,
  "all": 19
}
```

`tags.json` 通常包含：

```json
{
  "historyTags": [],
  "starredTags": []
}
```

`actions.json`、`saved-filters.json` 在样本库中是空数组，但结构上仍是图库顶层文件。

## 文件访问路径构造

```
库名.library/images/{id}.info/{name}.{ext}               ← 原图
库名.library/images/{id}.info/{name}_thumbnail.png        ← 缩略图
```

在代码中（使用 File System Access API）：

```javascript
// d = { id, name, ext, dn }  其中 dn = "{id}.info"
const dir = await imagesDir();                         // images/
const info = await dir.getDirectoryHandle(d.dn);       // images/{id}.info/
const file = await info.getFileHandle(d.name + '_thumbnail.png');  // 缩略图
const file = await info.getFileHandle(d.name + '.' + d.ext);       // 原图
```

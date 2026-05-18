# Eagle 图库目录结构

## 顶层结构

```
{库名}.library/
  metadata.json          ← 文件夹树结构（folders）
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
| `btime` | number | 创建时间戳（毫秒） | `1772248458713` |
| `mtime` | number | 修改时间戳（毫秒） | `1772248458716` |
| `palettes` | array | 调色板信息 | `[...]` |

## metadata.json（库级）

库根目录的 `metadata.json` 包含 `folders` 数组，描述文件夹树：

```json
{
  "folders": [
    {
      "id": "文件夹ID",
      "name": "文件夹名称",
      "children": [...]
    }
  ]
}
```

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

## 统计数据

- 总图片数：4558
- 缩略图缺失（无 `_thumbnail.png`）：26 张（0.6%），需 fallback 到原图
- 原图缺失：0 张
- `name` 为空：0 张

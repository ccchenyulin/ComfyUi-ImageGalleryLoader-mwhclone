# ComfyUI-ImageGalleryLoader

一个为 ComfyUI 设计的图像画廊自定义节点，提供可视化预览、多选、多源文件夹管理、子文件夹导航、剪贴板粘贴及删除等功能。

![预览图](https://github.com/user-attachments/assets/f84b5b26-ca14-4e81-ad66-17436ae98bad)

## 功能特性

- 🖼️ **可视化画廊** – 浏览电脑上任意文件夹中的图像。
- 📁 **多源文件夹** – 通过内置管理器添加/删除自定义图像文件夹。
- 📂 **子文件夹导航** – 层级浏览，动态下拉菜单。
- ✅ **多选** – 单击、Ctrl+单击、Shift+单击及范围选择。
- 🗑️ **删除图像** – 移至回收站（需安装 `send2trash`）或永久删除。
- 📋 **剪贴板粘贴** – 直接从剪贴板粘贴图像（Ctrl+V）。
- 🔍 **预览弹窗** – 双击或 Alt+单击图像查看原始分辨率。
- ⚡ **缓存与优化** – 缩略图和元数据缓存，提升性能。
- 🎛️ **排序与过滤** – 按名称/日期排序，按是否包含 ComfyUI 元数据过滤。
- 🎨 **可调节预览大小** – 滑块调整缩略图尺寸。
- 👁️ **自动隐藏预览** – 鼠标悬停时才显示图像（节省屏幕空间）。
- 🔁 **递归扫描** – 浏览时包含子文件夹。

## 安装方法

1. 进入 ComfyUI 的 `custom_nodes` 文件夹，打开终端。

2. 克隆本仓库：

   ```bash
   git clone https://github.com/BigStationW/ComfyUi-ImageGalleryLoader
   ```

3. 进入克隆后的文件夹：

   ```bash
   cd ComfyUi-ImageGalleryLoader
   ```

4. 安装所需 Python 依赖：

   ```bash
   ../../python_embeded/python.exe -s -m pip install -r requirements.txt
   ```

   > 如果使用系统 Python 或虚拟环境，请根据实际路径调整上述命令。

5. 重启 ComfyUI。

## 使用方法

重启后，在节点菜单的 **🖼️ Image Gallery** 分类下可以找到 **Image Gallery Loader** 节点。

### 基本操作流程

1. **添加节点** 到工作流中。
2. **选择源文件夹** 从下拉菜单中选取（或点击 “📁 Folder Manager” 添加更多文件夹）。
3. **浏览子文件夹** 使用动态导航栏（当存在子文件夹时出现）。
4. **单击图像** 进行选择 – 被选中的图像会有青色边框。
   - `Ctrl+单击` – 切换选中状态。
   - `Shift+单击` – 范围选择。
   - `Alt+单击` / 双击 – 打开预览弹窗。
5. **节点输出** 会将选中的图像以 **IMAGE 张量列表** 的形式输出（每个选中的图像对应一个张量）。  
   可使用 `Image Batch` 等节点将列表合并为批次。
6. **右键单击** 图像可弹出上下文菜单，包含：
   - 预览
   - 粘贴（从剪贴板）
   - 全选可见图像
   - 删除（单个或已选中的多个）

### 文件夹管理器

点击 **📁 Folder Manager** 按钮打开源文件夹管理对话框。

- **添加文件夹** – 点击 “Browse for Folder…” 并选择电脑上的任意目录。
- **移除文件夹** – 在列表中悬停到某个文件夹上，点击 “Remove”。  
  默认的 `input` 文件夹不可移除。
- 修改后立即保存，画廊会自动刷新。

### 子文件夹导航

当当前源文件夹包含子目录时，会出现水平导航栏：

- 每一级都是一个 **下拉菜单**，列出该层级的所有直接子文件夹。
- 选择某个文件夹可进入下一级；选择 “All” 则截断路径。
- 含有更深子文件夹的项会显示 `▸` 标记。
- 当前路径会 **持久化保存** – 重新打开工作流时会自动恢复。

### 快捷键

| 操作                     | 快捷键                         |
|--------------------------|--------------------------------|
| 全选可见图像             | `Ctrl+A`                       |
| 清空选择                 | `Escape`                       |
| 从剪贴板粘贴图像         | `Ctrl+V`（需要画廊容器获得焦点）|
| 关闭预览弹窗             | `Escape`                       |

> 画廊容器在点击内部时会自动获得焦点。

### 输出说明

节点返回一个 **列表**，其中每个元素是一个 `IMAGE` 张量，形状为 `(1, height, width, 3)`，对应一张选中的图像。  
如果没有选中任何图像，则返回一个空白的 64×64 张量（仅占位）。

若要同时处理多张图像，可以使用 ComfyUI 的 `Image Batch` 节点接收此列表。

## 配置文件

- **`custom_source_folders.json`** – 存储用户添加的源文件夹列表（位于节点目录内）。  
- **`image_gallery_ui_state.json`** – 持久化 UI 状态（当前源文件夹、子文件夹路径、排序/过滤设置、选中的图像等）。  
- **`thumbnail_cache/`** – 缓存 `.webp` 格式的缩略图，加快加载速度。

可以安全删除缓存文件夹，它会自动重建。

## API 端点（供开发者参考）

节点前端通过以下后端端点（位于 `/imagegallery/` 下）与后端通信：

| 端点                               | 方法   | 描述                         |
|------------------------------------|--------|------------------------------|
| `/get_source_folders`              | GET    | 获取配置的源文件夹列表。      |
| `/add_source_folder`               | POST   | 添加新文件夹。                |
| `/remove_source_folder`            | POST   | 移除文件夹。                  |
| `/browse_folder`                   | POST   | 打开系统文件夹选择对话框。     |
| `/get_images`                      | GET    | 分页获取图像及元数据。        |
| `/get_subfolders`                  | GET    | 获取直接子文件夹列表。        |
| `/thumb`                           | GET    | 提供缓存的缩略图（WebP）。     |
| `/preview`                         | GET    | 提供原始分辨率图像。          |
| `/delete_image`                    | POST   | 删除图像（移至回收站）。       |
| `/paste_image`                     | POST   | 接收剪贴板图像并保存。        |
| `/invalidate_cache`                | POST   | 清除内部缓存。                |
| `/set_ui_state` / `/get_ui_state`  | POST/GET | 保存/恢复 UI 状态。          |

## 依赖要求

- ComfyUI（建议最新版本）
- Python 包（见 `requirements.txt`）：
  - `Pillow`
  - `numpy`
  - `torch`
  - `send2trash`（可选但推荐，启用回收站支持）

## 常见问题及解决方法

### Linux 下文件夹选择器无法弹出

节点会尝试使用 `zenity` 或 `kdialog`（若 `tkinter` 不可用）。安装 `zenity`（GNOME）或 `kdialog`（KDE）即可使用原生文件夹对话框。  
也可以手动编辑 `custom_source_folders.json` 添加文件夹路径。

### 缩略图不显示 / 显示空白图像

- 检查图像文件是否损坏。
- 确认源文件夹已正确添加且可访问。
- 尝试点击 **🔄 Refresh** 按钮 – 这会清除缓存并重新扫描。

### 删除图像失败

如果没有安装 `send2trash`，图像会被永久删除（不经过回收站）。  
可通过 `pip install send2trash` 安装后重启 ComfyUI 来启用回收站功能。

### 粘贴功能提示 “Clipboard API not available”

- 使用 **Ctrl+V** 快捷键（画廊容器获得焦点时）进行粘贴 – 此方式使用原生剪贴板事件，始终有效。  
- 右键菜单中的 “Paste” 选项需要较新的 `navigator.clipboard.read()` API，某些浏览器或权限设置下可能不可用。

---

*在 ComfyUI 中轻松管理和选择您的图像！*

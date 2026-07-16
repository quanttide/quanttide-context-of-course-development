# 发布

本节介绍如何手动完成一次版本发布。如果你使用量潮DevOps云命令行工具，这些步骤会自动执行。

## 核心理念：小步快跑

发布不求大、不求全，有改动就尽早发。每个版本只包含少数几个变更，降低单次发布风险，也能让团队更快拿到反馈。

小步快跑不是降低质量，而是把大版本拆成多个小版本，每个小版本走完完整的发布流程。一个功能可以分两次发：先发后端接口（`v0.2.0`），再发前端页面（`v0.2.1`）。修复 bug 不必攒到下次大版本，修完就发（`v0.2.2`）。

预发布版本（alpha、beta、rc）也是小步快跑的具体实践——每个阶段都发一个版本标记，不等所有工作完成才打标签。

## 版本号规则

遵循语义化版本规范（SemVer）。正式版格式 `vX.Y.Z`，预发布版按开发阶段依次使用 `-alpha.N`、`-beta.N`、`-rc.N` 后缀。

## 操作前检查

确认以下事项后再执行：

- 最新标签与配置文件版本一致
- CHANGELOG 条目已存在
- 工作区干净（无未提交变更）

## 操作流程

### 1. 更新版本号

修改配置文件中的版本字段。

以 Python 项目为例（`pyproject.toml`）：

```diff
- version = "0.1.0"
+ version = "0.2.0"
```

### 2. 更新 CHANGELOG

在 `CHANGELOG.md` 顶部添加当前版本的变更记录：

```markdown
## [0.2.0] - 2026-06-29

### Added
- 新功能描述

### Fixed
- 修复描述
```

格式遵循 Keep a Changelog 规范。变更按 Added、Changed、Fixed、Removed 分类。

### 3. 提交修改

```bash
git add pyproject.toml CHANGELOG.md
git commit -m "chore: bump to v0.2.0"
```

### 4. 打标签

```bash
git tag v0.2.0
git push origin v0.2.0
```

标签格式为 `vX.Y.Z`（单组件项目）或 `scope/vX.Y.Z`（多组件项目，如 `cli/v0.6.0`）。

### 5. 创建 GitHub Release

Release body 须包含该版本的 CHANGELOG 内容：

```bash
gh release create v0.2.0 \
  --title v0.2.0 \
  --notes "CHANGELOG 内容粘贴到这里"
```

---

# 异常情况

## 处理规则

以下四条核心假设指导所有异常情况下的判断。

### Tag 不可移动

Tag 是发布过程中唯一不可重写的锚点。CHANGELOG 和 Release 都是派生制品，可以重建或修复，但 tag 一旦推送就不该动——动了会影响所有拉过这个版本的人。

异常处理从不敢建议移动 tag：补 Release、补 CHANGELOG、确认误推后删除——都在保护 tag 的不可变性。

### CHANGELOG 是规范事实源

CHANGELOG 是人工维护的结构化文档，有分类、有上下文、有格式约束。Release body 来源不定（自动生成 / AI 写 / 随手写），质量不可控。

因此：
- 缺 Release 时敢用 CHANGELOG 补（安全，事实源不变）
- 缺 CHANGELOG 时不能从 Release body 反推（不可靠，以 git log 为准）

### 只补全，不捏造

操作的极限是恢复一致性，不会创造新信息。没有事实源时宁可搁置，也不盲目补全。

异常处理的三个允许动作：
- **补建** — 从既有事实源重建缺失的派生制品
- **清理** — 移除废弃或错误的制品
- **搁置** — 信息不足时标记暂不处理

### 关系不可逆

发布生命周期的依赖方向：

```
commit → tag → CHANGELOG → Release
```

异常处理的方向是逆向追溯——缺什么就从左侧最近的事实源开始补。

## 一、缺 GitHub Release，但 tag + CHANGELOG 齐备

这种情况最常见，通常是发布时忘了执行 `gh release create`，或者 CI 的发布步骤失败了。

**处理方式**：用 CHANGELOG 内容补建 Release。

```bash
gh release create v0.2.0 \
  --title v0.2.0 \
  --notes "$(cat CHANGELOG.md 中 v0.2.0 版本的条目)"
```

这是最安全的修复——tag 和 CHANGELOG 都是已确认的事实源，补建 Release 不改变任何已有信息。

## 二、缺 CHANGELOG，但 tag + Release 存在

Release body 和 CHANGELOG 哪个更可信，取决于 Release body 的来源：

- **GitHub 自动生成的 Release notes**（从 commit 消息粗排）— 质量偏低，丢失分类、混入 chore，不适合直接回写。
- **AI 写的 Release notes** — 可能已经包含了结构化的变更描述，质量不一定差，但和 CHANGELOG 的格式规范（Keep a Changelog）可能有出入。
- **人工随手写的** — 可能不够完整或规范。

总之，Release body 的真实质量要 case by case 判断，不能一概否定。

**处理方式**：以 git log 为源补写 CHANGELOG，同时可以参考 Release body 的内容辅助判断变更分类。

```bash
git log v0.1.0..v0.2.0 --oneline
```

对照 commit 记录手动整理变更分类，遵循 Keep a Changelog 规范写入 CHANGELOG.md。没有捷径。

## 三、只有 tag，无 CHANGELOG 也无 Release

这是最棘手的情况——tag 只是一个指针，不携带任何发布元数据。没有事实源就谈不上"补全"。

**三个选择**：

1. **确认为误推**（比如开发途中不小心打上去的标签）→ 删除 tag：
   ```bash
   git push --delete origin v0.2.0
   git tag -d v0.2.0
   ```

2. **确认为有效发布**（确实完成了发布动作但只打了 tag）→ 跑一次完整的发布流程，让工具自动补全 CHANGELOG 和 Release。
   使用自动化工具：
   ```bash
   qtcloud-devops release publish -v v0.2.0 -y
   ```

3. **无法确认来源**（比如从同事遗留的分支上发现）→ 标记为"孤立 tag"，暂不处理，等有人能确认后再决定。

## 四、多出无关的 Release / tag

一些旧版本废弃后，对应的 Release 或 tag 还留在仓库中，会污染版本列表。

**处理方式**：清理。

```bash
# 删除 tag
git push --delete origin v0.1.0

# 删除 GitHub Release
gh release delete v0.1.0 --yes
```

## 五、Tag scope 前缀缺失

Tag 属于某个 scope（如 `cli`）但漏写了 scope 前缀，被打成根级别 tag。例如 `v0.1.0-rc.1` 应为 `cli/v0.1.0-rc.1`。这会导致自动化扫描误认为它是一个根级别发布，从而要求根 scope 有对应的 CHANGELOG 和 Release，产生假阳性。

**识别方式**：检查该 tag 对应的 commit 是否实际属于某个 scope 子目录。

**处理方式**：

1. 删除错误的根级别 tag：
   ```bash
   git tag -d v0.1.0-rc.1
   git push --delete origin v0.1.0-rc.1
   ```

2. 在正确的位置重建 tag（指向同一 commit）：
   ```bash
   git tag cli/v0.1.0-rc.1 <commit-sha>
   git push origin cli/v0.1.0-rc.1
   ```

3. 如果该 tag 已有对应的 Release（GitHub Release 创建时用了不带前缀的 tag 名），还需要删除孤立的 Release：
   ```bash
   gh release delete v0.1.0-rc.1 --yes
   ```

4. 补建正确的 Release：
   ```bash
   gh release create cli/v0.1.0-rc.1 --title "cli/v0.1.0-rc.1" --notes "..."
   ```

## 七、废弃的 Draft Release 污染

早期开发阶段创建的 Draft Release 不遵循 scope 前缀约定（如 `0.1.0-beta.1`、`0.1.0-beta.2`），且可能缺少 `v` 前缀。这些 Draft Release 没有对应的 tag，但会被自动化扫描误认为是根级别发布。

**识别方式**：`gh release list` 中标记为 `Draft` 的老版本。

**处理方式**：

```bash
gh release delete 0.1.0-beta.1 --yes
gh release delete 0.1.0-beta.2 --yes
```

如果对应的 tag 也存在，一并删除：

```bash
git tag -d 0.1.0-beta.1
git push --delete origin 0.1.0-beta.1
```

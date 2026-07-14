# 产品方向

## 客户场景的两极

两个真实案例暴露了产品需要同时服务两种极端场景：

| 维度 | Student（一对一） | University（校企） |
|------|---------|------------|
| 课程来源 | 按需生长，课后补结构 | 预先规划，先搭骨架 |
| 教学方式 | 实操为主，讲师引导 | 理论授课 + 实践 |
| 考核 | 二元通过/不通过 | 连续评分 + 评分标准 |
| 班级规模 | 1 人 | 40 人 |
| 教师 | 1 人 | 多人（企业 + 校内） |
| 节奏 | 灵活，按学员进度 | 固定学期制 |

## 产品当前的隐性偏向

当前产品设计在无意识地偏向 university 模式：

- 六级树天然适合自上而下规划
- ContentStatus 控制发布流程（适合有版本管理的正式课程）
- Class 模型预设了多人教务管理场景
- Assessment 模型（maxScore + passingScore）更接近连续评分

Student 场景需要的"先有叶子再长树干"的自下而上路径，产品有规划（独立课时池 + 归入阶段）但尚未实现。

## Studio 不是编辑器，是视图切换器

不同客户看到的内容结构和交互方式不同。同样的底层数据（六级树 + Class + Assessment）通过不同的视图组合呈现：

| 角色 | 关注点 | Studio 视图 |
|------|--------|------------|
| 个人学员 | 本节课任务、作业 | 看板模式 |
| 讲师 | 课程结构、学生进度 | 树 + 考核列表 |
| 教务 | 完整六级树 + 班级管理 | 树 + 表格 |

如果角色不做，直接做 class 和 assessment 功能，会在 UI 上强行把两种用户的视图塞进同一个布局里。

## 课程研发和教学实施之间的墙会塌

三领域隔离（课前/课中/课后）在数据层是对的，但 Studio 的体验不应反映这个隔离。用户不关心他在哪个领域界面，他只想完成"上这节课"这个任务。

导航设计应逐步向用户任务靠拢：

```
今天的任务流
├── 备课（查看/调整本节课内容）
├── 上课（播放场景/步骤引导）
├── 检查作业（查看未评分提交）
└── 记录（更新课程内容）
```

底层数据仍然是课前/课中/课后隔离的，但 Studio 的 UI 层为用户组装这些领域。

## 考核模型需要评分模式切换

- Student：二元考核，只需要 passed（true/false）
- University：连续评分，需要 maxScore、score、评分标准

当前 Assessment 模型更适合 University。建议支持 `gradingMode: pass_fail | numeric`。pass_fail 模式下隐藏 maxScore、score 等字段，只显示 passed。

## 对 ROADMAP 的影响

v0.0.5 影响不大——课程编辑是底层能力，不管什么视图都需要。

v0.0.6 如果要同时做 class + assessment，建议先定义角色系统（角色 → 视图 → 权限），否则 UI 会为兼顾两种用户而变得臃肿。

## 技术实现细节

以下为从 intention 仓库降级至此的实现级决策。

### 蓝图互通

- 蓝图（JSON）是 CLI 和 Studio 的共同数据格式
- 冲突处理策略：覆盖/跳过/保留两者，保护人工劳动成果

### 发布安全

- 内容状态只有 `draft` 和 `published` 两种
- 编辑未保存有标记
- 删除需确认并提示子级数量
- 已发布内容不可直接删除，必须先下架

### 数据模型

领域：

- 课前（课程研发）：Program → Course → Phase → Lesson → Scene → Step，ContentStatus 控制发布
- 课中（教学实施）：Class + Teacher + Student，ClassStatus 控制教学流程
- 课后（考核评估）：Assessment + Submission + Grading，AssessmentStatus 控制考核流程

考核：

- `gradingMode: pass_fail | numeric` 切换评分模式
- pass_fail 模式只显示 passed（true/false），隐藏 maxScore、score
- numeric 模式显示 maxScore、passingScore、score、评分标准

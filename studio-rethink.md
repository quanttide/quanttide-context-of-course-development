# 从客户案例反观 Studio 设计

结合 gallery 中两个案例（student / university）对 Studio 的重新思考。

## 1. Studio 不是编辑器，而是"视图切换器"

之前把 Studio 想象成课程编辑器——用户进来编辑六级树。但不同客户看到的内容结构和交互方式应该不同：

- University 模式：六级树是自然的——先定 Program → Course → Phase → Lesson → Scene → Step
- Student 模式：六级树是困惑的——他只有 1 个 Program、1 个 Course，他关心的是"这节课我该做什么"

**设想**：同样的底层数据（六级树 + Class + Assessment），通过不同的视图组合呈现。底层 Service 不变，上层 Widget 按角色组装。不是写死两个版本，而是数据模型通过不同的组合方式呈现。

## 2. 课程研发和教学实施之间的墙会塌

Student 案例中，讲师上课时也会修改课程内容（课后补结构），考核结果也会影响下一节课的内容。

三领域隔离在数据层是对的，但 Studio 作为客户端的体验不应该反映这个隔离。用户不关心他是在"课前界面"还是"课中界面"，他只想完成"上这节课"这个任务。

导航设计不应按领域来组织，而要按**用户任务**来组织：

```
今天的任务流
├── 备课（查看/调整本节课内容）
├── 上课（播放场景/步骤引导）
├── 检查作业（查看未评分提交）
└── 记录（更新课程内容）
```

这不是否定领域架构——底层数据仍然是课前/课中/课后隔离的。但 Studio 的 UI 层应该为用户组装这些领域，而不是暴露领域边界。

## 3. 考核模型需要评分模式切换

- Student：二元考核（通过/不通过）
- University：连续评分 + 评分标准模板

当前 Assessment 模型（maxScore + passingScore + score）更适合 University。如果强行塞进 Student 场景会过度工程化。

**设想**：Assessment 支持 `gradingMode: pass_fail | numeric`。pass_fail 模式下隐藏 maxScore、passingScore、score 等字段，只显示 passed（true/false）。

## 4. 对 ROADMAP 的影响

v0.0.5 影响不大——课程编辑是底层能力，不管什么视图都需要。

v0.0.6 的原规划是"教学与考核管理"。现在看可能应该改为：**v0.0.6 先定义 Studio 的用户角色系统（角色 → 视图 → 权限），再做功能。**

不同的角色关注点和 Studio 视图：

| 角色 | 关注点 | Studio 视图 |
|------|--------|------------|
| 个人学员（Student） | 本节课任务、作业 | 看板模式 |
| 讲师（Teacher） | 课程结构、学生进度 | 树 + 考核列表 |
| 教务（University） | 完整六级树 + 班级管理 | 树 + 表格 |

如果角色不做，直接做 class 和 assessment 功能，就会在 UI 上强行把两种用户的视图塞进同一个布局里。

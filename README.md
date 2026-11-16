# Repo B · 中国海洋大学天文协会（移动端）

中国海洋大学天文协会「星海课堂」移动端。与桌面端星环中枢仓库完全物理隔离，但共用同一个 CloudBase 环境和同一套 `members` 成员账号。

## 信息架构

底部导航固定为五项，按角色显隐：

| 导航 | 页面 | 内容 | 可见性 |
| --- | --- | --- | --- |
| 工作台 | 当前课程、签到、实时提问、星海漫游 | 课次与课堂工具 | 所有成员 |
| 我的课程 | 课程列表、加入课程 | `courseService.list_courses` | 所有成员 |
| 学习资源 | 视频教程 | `videoService.list` | 所有成员 |
| 实验数据 | 光变实验记录与曲线 | `get_classroom_state.experiments` | 所有成员 |
| 控制台 | 成员、待审核、课程、签到概览 | `memberService.admin_list` 等 | 仅管理员 / 最高权限 |

- 普通成员看不到「控制台」
- 签到、实时提问、星海漫游作为工作台内的课堂工具，不单独占用导航
- 每个页面统一结构：面包屑 `中国海洋大学天文协会 / 页面名` → `Kicker` → `H1` → 副标题 → 分区 `H2`
- 个人档案从头像进入，包含「技术支持：星环终端」

## 文件

```text
index.html   # 唯一入口，内联 CSS/JS，无需构建
.nojekyll
README.md
```

## 成员账号：与星环中枢共用

登录直接调用 `memberService.web_login`：

```js
const res = await app.callFunction({
  name: 'memberService',
  data: { action: 'web_login', username, password }
});
const { ticket, token, user } = res.result.data;
await auth.signInWithCustomTicket(() => Promise.resolve(ticket));
```

- 使用已注册的星环成员账号，**无需重复注册**
- 角色、社团、部门来自同一条 `members` 记录
- 需要 `memberService` 已配置自定义登录私钥

## 页面数据

| 页面 | 调用 |
| --- | --- |
| 工作台 | `courseService.list_courses` → `get_classroom_state` |
| 签到 | `courseService.submit_checkin` |
| 提问 | `courseService.send_classroom_chat` + `classroom_chats.watch()` |
| 我的课程 | `courseService.list_courses` / `join_course` |
| 学习资源 | `videoService.list` |
| 实验数据 | `get_classroom_state.experiments` |
| 控制台 | `memberService.admin_list` / `admin_review_list`、`courseService.list_courses` |

## 扫码签到

桌面端二维码会带上：

```text
?course_id=<course_id>&session_id=<session_id>&org=<org>&code=<code>
```

移动端打开后：

1. 在登录页显示「扫码已识别」；
2. 登录后按 `course_id` 选择课程；
3. 进入工作台，老师开启签到后点击签到；
4. 签到写入当前课次 `course_sessions.phase_payload.checkin`。

签到码为辅助展示；实际校验以 `courseService.submit_checkin` 的课次状态为准。

## 星海漫游

- 纯 Canvas，无三方星图库
- 拖动平移、滚轮缩放、点击星点显示星座信息
- 内置北斗七星、猎户座、仙后座、夏季大三角
- 星图交互由星环终端提供技术支持

## 部署到 GitHub Pages

1. 新建仓库，例如 `repo-b-ouc-astronomy`；
2. 推送 `index.html`、`.nojekyll`、`README.md`；
3. Settings → Pages → Deploy from a branch → root；
4. 把 Pages 地址填入 Repo A 的 `CONFIG.studentAppUrl`；
5. 在 CloudBase 配置 Web 安全域名与数据库规则。

## 演示模式

未登录时可点击「体验演示」查看五页 IA、签到、提问、星海漫游与控制台。演示数据不会写入云端。

## 技术支持展示

- 登录页页脚
- 工作台页脚
- 学习资源页脚
- 实验数据页脚
- 控制台页脚
- 星海漫游底部
- 个人档案技术支持卡片

统一文案：**中国海洋大学天文协会 · 技术支持：星环终端**

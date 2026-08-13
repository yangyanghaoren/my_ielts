# English Prep Lab

English Prep Lab 是一个面向英语备考的个人练习站点，重点放在高频词汇、听力替换、阅读同义表达、写作句型和语法复习。项目已经整理成可直接部署到 GitHub Pages 的 Vue 单页应用。

在线地址：[https://yangyanghaoren.github.io/my_ielts/#/](https://yangyanghaoren.github.io/my_ielts/#/)

## 功能

- 主题词汇训练：分类筛选、关键词搜索、音频播放、词义/例句查看。
- 听写练习：输入拼写、自动判错、错词复制、按掌握类型筛选。
- 单词跟打：按分类练习输入速度和单词熟悉度。
- 听力、阅读、写作、语法：保留常用复习资料和练习入口。
- 深色模式：支持本地记忆显示偏好。

## 技术栈

- Vue 3
- Vue Router
- Vite
- UnoCSS
- VueUse

## 本地开发

```bash
pnpm i
pnpm run dev
```

默认开发地址：

```text
http://127.0.0.1:3333/
```

## 构建

```bash
pnpm run build
```

如在 Windows 上遇到 `dist` 清理占用问题，可以先用下面命令做编译验证：

```bash
node_modules\.bin\vite.cmd build --emptyOutDir false
```

## 部署

当前仓库通过 GitHub Actions 从 `dev` 分支构建，并发布到 GitHub Pages。推送到 `dev` 后会自动触发部署。

## 说明

本项目仅用于个人学习和练习，不用于商业用途。

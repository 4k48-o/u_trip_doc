# 变更日志 (Changelog)

> 遵循 [Keep a Changelog](https://keepachangelog.com/) 规范。

---

## 版本规范

版本号格式：`MAJOR.MINOR.PATCH`

| 类型 | 说明 |
|------|------|
| MAJOR | 不兼容的 API 变更 |
| MINOR | 向后兼容的功能新增 |
| PATCH | 向后兼容的 Bug 修复 |

---

## [Unreleased]

### Added
- （新增的功能）

### Changed
- （变更的现有功能）

### Deprecated
- （即将废弃的功能）

### Removed
- （已移除的功能）

### Fixed
- （已修复的 Bug）

### Security
- （安全相关的修复）

---

## [0.1.0] - YYYY-MM-DD

### Added
- 初始化项目

---

## 发布流程

1. 在 develop 分支修改 CHANGELOG.md，将变更填入 `[Unreleased]`
2. 发布时，将 `[Unreleased]` 替换为版本号和日期
3. 打 Git Tag：`git tag -a v0.1.0 -m "Release v0.1.0"`
4. Tag 推送后触发 CI 发布

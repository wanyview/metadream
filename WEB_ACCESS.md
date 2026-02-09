# Metadream 网页访问方式

## 方式一：本地直接打开（立即可用）

双击文件直接访问：
```
projects/dreamlearn-system/index.html
```

## 方式二：GitHub Pages（在线访问）

GitHub Pages会自动部署，访问地址：
```
https://wanyview.github.io/metadream/
```

（首次推送后约2-5分钟生效）

## 方式三：使用VS Code Live Server

```bash
# 在项目目录运行
npx http-server . -p 3000
# 访问 http://localhost:3000
```

## 网页功能

- ✅ 交互式睡眠状态监测演示
- ✅ 脑波实时可视化
- ✅ REM诱导触发演示
- ✅ 梦境学习流程
- ✅ 学习报告生成

## 快速测试

1. 双击打开 `index.html`
2. 点击「开始监测」模拟睡眠流程
3. 等待自动切换到REM阶段
4. 点击「触发REM诱导」
5. 点击「开始梦境学习」
6. 点击「生成学习报告」

## 技术文档

完整系统设计文档：
```
README.md
01-系统总架构.md
02-核心数据流图.md
03-模块与API清单.md
...
```

GitHub仓库：https://github.com/wanyview/metadream

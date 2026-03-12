# MPPI 批评函数代价计算器

一个交互式的在线工具，用于计算和分析 MPPI (Model Predictive Path Integral) 控制器中各批评函数的代价值。

## 🌟 在线访问

**网址**: [https://mppi-cost-calculator.vercel.app](https://mppi-cost-calculator.vercel.app)

## ✨ 功能特性

### 📱 实时计算器
- 10个批评函数的实时计算
- 数学公式展示（LaTeX风格）
- 代价分解可视化
- 自动高亮最大代价项

### 📊 可视化分析
- 代价随参数变化曲线
- 各批评函数代价占比饼图
- 支持选择不同批评函数分析

### 🔍 敏感性分析
- 参数变化对总代价的影响
- 可视化敏感度条形图
- 帮助识别关键参数

### 💾 配置管理
- 导出/导入 JSON 格式配置
- 导出/导入 YAML 格式配置
- 预设场景快速切换
- 生成详细分析报告

## 🚀 本地运行

```bash
# 进入项目目录
cd docs

# 安装依赖
npm install

# 启动本地服务器
npm start
```

然后在浏览器中访问 `http://localhost:3000`

## 📦 部署

本项目使用 Vercel 进行部署：

```bash
# 安装 Vercel CLI
npm i -g vercel

# 登录 Vercel
vercel login

# 部署
vercel --prod
```

## 📝 批评函数列表

1. **ObstaclesCritic** - 障碍物代价
2. **PathFollowCritic** - 路径跟随代价
3. **PathAlignCritic** - 路径对齐代价
4. **PathAngleCritic** - 路径角度代价
5. **GoalCritic** - 目标点代价
6. **GoalAngleCritic** - 目标角度代价
7. **PreferForwardCritic** - 偏好前进代价
8. **ConstraintCritic** - 约束代价
9. **TwirlingCritic** - 旋转惩罚代价
10. **VelocityDeadbandCritic** - 速度死区代价

## 📄 相关文件

- [critic_cost_formulas.md](critic_cost_formulas.md) - 详细的数学公式说明

## 🔗 相关链接

- [MPPI Controller GitHub](https://github.com/your-repo/mppi-controller)
- [Vercel 部署文档](https://vercel.com/docs)

## 📄 License

MIT License

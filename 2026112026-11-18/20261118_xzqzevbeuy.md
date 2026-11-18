# 聊聊 Docker 镜像瘦身

> 镜像体积直接影响拉取速度和攻击面，能删的东西就不要留在最终层。

## 为什么值得关注

这段时间在项目里反复用到Docker 镜像瘦身，踩过一些坑之后，总结出一套自己用起来比较顺的做法。

## 核心要点

- 合并 RUN 指令并及时清理包管理器缓存。
- 用 .dockerignore 排除无关文件，避免把本地目录整个塞进去。
- 基础镜像优先选 slim 或 alpine，但要留意 glibc 兼容性。
- 多阶段构建：编译环境和运行环境分开，只把产物带进最终镜像。

## 具体做法

下面用一个最小示例把流程串起来，实际项目里按需扩展即可：

```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip wheel --no-cache-dir -r requirements.txt -w /wheels

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /wheels /wheels
RUN pip install --no-cache-dir /wheels/* && rm -rf /wheels
COPY . .
CMD ["python", "app.py"]
```

如果没有现成的测试环境，可以先在小范围数据上验证，再逐步放量。

## 注意事项

- 注意构建缓存顺序：变化最少的层放在最前面。
- 生产镜像尽量使用非 root 用户运行。

## 小结

Docker 镜像瘦身没有银弹，关键是把取舍想清楚，然后保持记录和复盘。

---
*2026年11月18日*

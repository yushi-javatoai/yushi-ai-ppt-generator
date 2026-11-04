# 领域文档

工程类技能在探索代码库时，应如何使用本仓库的领域文档。

## 探索前先读这些

- 仓库根目录的 **`GLOSSARY.md`**，或
- 若根目录存在 **`GLOSSARY-MAP.md`**：它指向每个上下文各自的 `GLOSSARY.md`。只读与当前主题相关的那些。
- **`docs/adr/`**：阅读即将改动区域相关的 ADR。多上下文仓库还要查看 `src/<context>/docs/adr/` 中该上下文范围内的决策。

若这些文件不存在，**直接继续**。不要指出缺失，也不要事先建议创建。`/domain-modeling` 技能（经由 `/grill-with-docs` 与 `/improve-codebase-architecture` 进入）会在术语或决策真正敲定时再按需创建。

## 文件结构

单上下文仓库（多数仓库）：

```
/
├── GLOSSARY.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

多上下文仓库（根目录存在 `GLOSSARY-MAP.md`）：

```
/
├── GLOSSARY-MAP.md
├── docs/adr/                          ← 全系统决策
└── src/
    ├── ordering/
    │   ├── GLOSSARY.md
    │   └── docs/adr/                  ← 该上下文的决策
    └── billing/
        ├── GLOSSARY.md
        └── docs/adr/
```

## 使用术语表中的用词

输出里提到领域概念时（Issue 标题、重构提案、假设、测试名），使用 `GLOSSARY.md` 中定义的术语。不要改用术语表明确避开的同义词。

若需要的概念还不在术语表里，这是一个信号：要么你在发明项目并未使用的说法（应重新考虑），要么确实存在缺口（记下来交给 `/domain-modeling`）。

## 标出与 ADR 的冲突

若输出与已有 ADR 矛盾，要明确指出，不要悄悄覆盖：

> _与 ADR-0007（event-sourced orders）矛盾，但值得重新讨论，因为……_

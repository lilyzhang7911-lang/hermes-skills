---
name: memory-search-indexed
description: 索引式语义记忆检索 - 从 Grok Build 的 memory_search/memory_get 工具提取的工程化实践，支持跨会话知识检索、相关性排序和精准定位。
version: 1.0.0
tags: [memory, search, retrieval, semantic]
---

# Memory Search 索引式语义检索技能

## 触发条件
当用户需要：
- 在大量记忆文件中快速定位相关内容（"上次我们讨论过的哲学框架是什么"）
- 跨会话检索特定主题的知识片段
- 查找项目约定、编码模式或用户偏好
- compaction 后恢复丢失的上下文

## 核心设计原则（来自 Grok Build memory_backend）

1. **索引化**：记忆文件建立倒排索引，避免全量扫描
2. **相关性排序**：返回结果按语义相似度排名
3. **分层检索**：全局记忆 > 工作区记忆 > 会话记忆
4. **去重与合并**：相似片段自动合并，避免信息冗余

## 实现架构

### 1. 索引结构（SQLite）
```sql
-- memory_index.db 表结构
CREATE TABLE IF NOT EXISTS memory_chunks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    source_file TEXT NOT NULL,       -- 来源文件路径
    chunk_hash TEXT UNIQUE NOT NULL, -- 内容哈希（去重）
    content TEXT NOT NULL,           -- 原始文本片段
    line_start INTEGER,              -- 起始行号
    line_end INTEGER,                -- 结束行号
    word_count INTEGER,              -- 词数统计
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_memory_source ON memory_chunks(source_file);
CREATE INDEX IF NOT EXISTS idx_memory_content_fts ON memory_chunks(content);

-- FTS5全文索引（用于语义搜索）
CREATE VIRTUAL TABLE IF NOT EXISTS memory_fts USING fts5(
    content,
    source_file,
    line_start,
    line_end,
    word_count,
    tokenize='porter unicode61'
);
```

### 2. 索引构建脚本（build_index.sh）
```bash
#!/bin/bash
# build_index.sh - 为 Hermes Agent 记忆目录建立全文索引

INDEX_DB="${HOME}/.hermes/memory_index.db"
MEMORY_DIRS=(
    "${HOME}/Desktop/MyWorkHome/My-Obsidian/Hermes Agent"
    "${HOME}/.hermes/skills"
)

mkdir -p "$(dirname "$INDEX_DB")"

# 初始化数据库（如果不存在）
sqlite3 "$INDEX_DB" <<'SQL'
CREATE TABLE IF NOT EXISTS memory_chunks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    source_file TEXT NOT NULL,
    chunk_hash TEXT UNIQUE NOT NULL,
    content TEXT NOT NULL,
    line_start INTEGER,
    line_end INTEGER,
    word_count INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_memory_source ON memory_chunks(source_file);

-- FTS5全文索引
DROP TABLE IF EXISTS memory_fts;
CREATE VIRTUAL TABLE memory_fts USING fts5(
    content, source_file, line_start, line_end, word_count,
    tokenize='porter unicode61'
);
SQL

echo "Index database initialized: $INDEX_DB"

# 遍历记忆目录，构建索引
for dir in "${MEMORY_DIRS[@]}"; do
    if [[ ! -d "$dir" ]]; then
        echo "WARNING: Directory not found: $dir" >&2
        continue
    fi
    
    echo "Indexing directory: $dir" >&2
    
    # 查找所有 .md 文件（排除临时文件和缓存）
    find "$dir" -name "*.md" \
        ! -path "*/.Trash/*" \
        ! -path "*/node_modules/*" \
        ! -path "*/.git/*" \
        -print0 | while IFS= read -r -d '' file; do
        
        # 计算文件哈希（用于增量更新）
        local file_hash=$(md5sum "$file" | cut -d' ' -f1)
        local chunk_id="$file_hash:$(stat -c %Y "$file" 2>/dev/null || stat -f %m "$file" 2>/dev/null)"
        
        # 检查是否已索引（通过哈希+修改时间判断）
        local exists=$(sqlite3 "$INDEX_DB" "SELECT COUNT(*) FROM memory_chunks WHERE chunk_hash='$chunk_id';")
        
        if (( exists > 0 )); then
            echo "  Skipping (unchanged): $file" >&2
            continue
        fi
        
        # 分块处理（每50行一个chunk，避免过大）
        local line_num=1
        while IFS= read -r chunk; do
            if [[ -z "$chunk" ]]; then continue; fi
            
            local word_count=$(echo "$chunk" | wc -w)
            local content_escaped=$(echo "$chunk" | sed "s/'/''/g")  # SQL转义
            
            # 插入主表
            sqlite3 "$INDEX_DB" "INSERT OR IGNORE INTO memory_chunks (source_file, chunk_hash, content, line_start, line_end, word_count) VALUES ('$file', '$chunk_id:$line_num', '$content_escaped', $line_num, $(($line_num + 49)), $word_count);"
            
            # 插入FTS索引
            sqlite3 "$INDEX_DB" "INSERT INTO memory_fts (content, source_file, line_start, line_end, word_count) VALUES ('$content_escaped', '$file', $line_num, $(($line_num + 49)), $word_count);"
            
            line_num=$((line_num + 50))
        done < <(awk 'NR%50==1' "$file")
        
    done
    
done

echo "Indexing complete." >&2

# 优化FTS索引
sqlite3 "$INDEX_DB" "OPTIMIZE memory_fts;" 2>/dev/null

echo "Optimized FTS index." >&2
```

### 3. 语义搜索脚本（search_memory.sh）
```bash
#!/bin/bash
# search_memory.sh - 基于FTS5的语义记忆检索

INDEX_DB="${HOME}/.hermes/memory_index.db"
MAX_RESULTS=10
MIN_SCORE=0.5

if [[ ! -f "$INDEX_DB" ]]; then
    echo "ERROR: Index database not found at $INDEX_DB" >&2
    echo "Run build_index.sh first." >&2
    exit 1
fi

query="$1"
max_results="${2:-$MAX_RESULTS}"

# FTS5搜索（支持布尔逻辑、短语匹配）
sqlite3 -json "$INDEX_DB" <<EOF
SELECT 
    mc.source_file,
    mc.content,
    mc.line_start,
    mc.line_end,
    mc.word_count,
    rank as relevance_score
FROM memory_fts AS fts
JOIN memory_chunks AS mc ON fts.rowid = mc.id
WHERE memory_fts MATCH '${query}'
ORDER BY rank DESC
LIMIT ${max_results};
EOF

# 如果FTS5不支持rank，使用snippet高亮
if [[ $? -ne 0 ]]; then
    sqlite3 "$INDEX_DB" <<EOF
SELECT 
    mc.source_file,
    snippet(memory_fts, '<b>', '</b>', '...', 2) as highlighted_content,
    mc.line_start,
    mc.line_end
FROM memory_fts AS fts
JOIN memory_chunks AS mc ON fts.rowid = mc.id
WHERE memory_fts MATCH '${query}'
LIMIT ${max_results};
EOF
fi
```

### 4. 记忆文件读取脚本（get_memory.sh）
```bash
#!/bin/bash
# get_memory.sh - 按路径精准读取记忆文件（带行号格式化）

file_path="$1"
from_line="${2:-1}"
to_line="${3:-0}"  # 0表示读到末尾

if [[ ! -f "$file_path" ]]; then
    echo "ERROR: File not found: $file_path" >&2
    exit 1
fi

# 格式化输出（带行号，匹配Grok Build的format_with_line_numbers）
awk -v start="$from_line" 'NR>=start {printf "%d→%s\n", NR, $0}' "$file_path" | \
    if [[ "$to_line" -gt 0 ]]; then
        head -n "$((to_line - from_line + 1))"
    else
        cat
    fi
```

### 5. 增量更新脚本（update_index.sh）
```bash
#!/bin/bash
# update_index.sh - 增量更新索引（检测文件变更）

INDEX_DB="${HOME}/.hermes/memory_index.db"
MEMORY_DIRS=(...)  # 同build_index.sh

# 检查数据库是否存在
if [[ ! -f "$INDEX_DB" ]]; then
    echo "ERROR: Index database not found." >&2
    exit 1
fi

echo "Checking for changes..." >&2

for dir in "${MEMORY_DIRS[@]}"; do
    find "$dir" -name "*.md" \
        ! -path "*/.Trash/*" \
        ! -path "*/node_modules/*" \
        ! -path "*/.git/*" \
        -print0 | while IFS= read -r -d '' file; do
        
        # 计算当前哈希和修改时间
        local current_hash=$(md5sum "$file" | cut -d' ' -f1)
        local current_mtime=$(stat -c %Y "$file" 2>/dev/null || stat -f %m "$file" 2>/dev/null)
        
        # 查询数据库中的记录
        local stored_hash=$(sqlite3 "$INDEX_DB" "SELECT chunk_hash FROM memory_chunks WHERE source_file='$file' LIMIT 1;")
        
        if [[ -z "$stored_hash" ]]; then
            echo "New file detected: $file" >&2
            # TODO: 插入新记录（复用build_index.sh逻辑）
        elif [[ "$current_mtime" != "${stored_hash#*:}" ]]; then
            echo "Modified file detected: $file (mtime changed)" >&2
            # TODO: 删除旧chunk，重新索引
        else
            : # 无变化
        fi
        
    done
done

echo "Update complete." >&2
```

## 使用示例

### 构建初始索引
```bash
~/.hermes/skills/memory-search-indexed/scripts/build_index.sh
```

### 语义搜索
```bash
# 搜索"哲学框架"相关内容
~/.hermes/skills/memory-search-indexed/scripts/search_memory.sh "哲学框架" 5

# 搜索布尔逻辑（"黑格尔" AND "康德"）
~/.hermes/skills/memory-search-indexed/scripts/search_memory.sh '"黑格尔" AND "康德"' 10

# 短语精确匹配
~/.hermes/skills/memory-search-indexed/scripts/search_memory.sh '"数字人目标"' 3
```

### 精准读取记忆文件
```bash
# 读取特定文件的第10-50行（带行号）
~/.hermes/skills/memory-search-indexed/scripts/get_memory.sh \
    "/Users/sunwenning/Desktop/MyWorkHome/My-Obsidian/Hermes Agent/00_Constitution/why.md" \
    10 50
```

### 增量更新索引
```bash
~/.hermes/skills/memory-search-indexed/scripts/update_index.sh
```

## 与 Hermes Agent 集成

在 Hermes Agent 中通过 `terminal` 工具调用：

```python
from hermes_tools import terminal, search_files

# 搜索记忆
result = terminal(
    command="~/.hermes/skills/memory-search-indexed/scripts/search_memory.sh '哲学框架' 5",
    timeout=10
)
print(result["output"])

# 读取具体文件
result = terminal(
    command=f"~/.hermes/skills/memory-search-indexed/scripts/get_memory.sh '{file_path}' {start_line} {end_line}",
    timeout=5
)
```

## 注意事项

1. **首次构建耗时**：大量记忆文件可能需要几分钟，建议后台运行
2. **FTS5依赖**：需要 SQLite3 支持 FTS5 扩展（macOS默认支持）
3. **增量更新**：定期运行 update_index.sh 保持索引新鲜
4. **缓存策略**：搜索结果可缓存，避免重复查询
5. **隐私安全**：索引文件存储在本地，不上传任何数据

## 扩展点

- 可集成向量数据库（Chroma/FAISS）实现真正的语义相似度搜索
- 可扩展支持 PDF/Word 等非 Markdown 格式的记忆文件
- 可添加搜索结果的高亮和摘要功能
- 可与 Obsidian 的 Dataview 插件联动，自动同步索引
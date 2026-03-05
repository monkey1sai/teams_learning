# Lessons

## 2026-03-05

- 使用者明確要求 `.venv` 不可加入版控時，必須：
  - 檢查 `.gitignore` 是否包含 `.venv/`
  - 用 `git ls-files` 確認沒有任何 `.venv/` 被追蹤
  - 在 README 提供 `uv` 重建環境步驟，避免把虛擬環境提交到 Git

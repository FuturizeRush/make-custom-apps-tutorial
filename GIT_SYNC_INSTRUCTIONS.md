# GitBook Git 同步設置

## 步驟 1: 創建 GitHub 倉庫
1. 前往 https://github.com/new
2. 創建一個新的倉庫（例如：make-custom-apps-tutorial）
3. 設為 Public 或 Private（根據需要）

## 步驟 2: 推送內容到 GitHub
```bash
cd gitbook-sync
git init
git add .
git commit -m "Initial commit: Make Custom Apps Tutorial"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

## 步驟 3: 在 GitBook 中設置 Git 同步
1. 訪問您的 GitBook 空間: https://app.gitbook.com/o/DyUpk4VO4veiBFr3yQnd/s/bljKl8LyshMkU9Um25OF/
2. 點擊右上角設定圖標
3. 選擇 "Git Sync" 或 "Git 同步"
4. 選擇 GitHub
5. 授權 GitBook 訪問您的 GitHub
6. 選擇剛創建的倉庫
7. 選擇 main 分支
8. 點擊 "Synchronize" 或 "同步"

## 完成！
GitBook 會自動從 GitHub 倉庫同步內容。之後您可以：
- 在 GitHub 上編輯內容，GitBook 自動更新
- 在 GitBook 上編輯，變更會推送到 GitHub
- 享受版本控制的所有好處

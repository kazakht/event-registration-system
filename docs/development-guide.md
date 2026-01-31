# AI駆動開発スタートガイド：GitHub PRフロー編

## 1. ローカルリポジトリの初期化

```powershell
git init
echo "node_modules/
.next/
.env
.DS_Store" > .gitignore
git add .
git commit -m "chore: プロジェクト初期化とドキュメントの追加"
```

## 2. GitHubリポジトリへの紐付け

```powershell
git remote add origin [コピーしたURL]
git branch -M main
git push -u origin main
```

## 3. プルリクエスト（PR）開発のルーチン

1. `git checkout -b feat/feature-name` でブランチ作成
2. 実装 & `git add .` & `git commit -m "message"`
3. `git push origin feat/feature-name`
4. GitHub上で Pull Request を作成してマージ
5. `git checkout main` & `git pull origin main` でローカルを同期

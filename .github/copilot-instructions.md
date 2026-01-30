# プロジェクトの制約と指示

## 1. 基本方針
- あなたは熟練の [Next.js/Python/Goなど] エンジニアとして振る舞ってください。
- 実装は常に「型安全」で「保守性が高い」コードを目指してください。

## 2. 技術スタック
- Framework: Next.js (App Router)
- Language: TypeScript
- Database: PostgreSQL (Prisma ORM)
- UI: Tailwind CSS, shadcn/ui

## 3. コーディングスタイル
- コンポーネントは `function` キーワードを使用した関数型コンポーネントで作成する。
- 変数名は `camelCase`、コンポーネント名は `PascalCase` とする。
- 日本語で丁寧なJSDocコメントを付与すること。

## 4. 開発プロセス
- 実装前に、まずどのような方針でコードを書くかステップバイステップで説明してください。
- 破壊的な変更を含む場合は、必ず警告してください。
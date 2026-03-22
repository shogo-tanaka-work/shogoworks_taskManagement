# タスク管理の運用ルール

## Linear × GitHub Projects の役割分担

- **Linear**: 人生・業務全体の進行管理（案件、営業、学習、プライベート）
- **GitHub Projects**: リポジトリ単位の開発イシュー管理（実装、バグ修正、CI/CD）

## どちらにタスクを作るか

- コードを書く → GitHub Issue
- コードを書かない → Linear
- 複数リポにまたがる → Linear（上位）+ GitHub Issue（実装単位）

## Linear プロジェクト構成

- byTech_タスク管理: 技術系ビジネスタスク
- 人生タスク管理: プライベート・学習

## GitHub Issue 運用

- Sprint自動割り当て: 期限を設定すれば assign-sprint.yml が自動処理
- カテゴリ: 機能追加 / バグ修正 / CI・インフラ / ドキュメント / リファクタ

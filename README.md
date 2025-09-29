# 申請承認ワークフローシステム

PHP Laravel + MariaDBで構築された簡易的な申請承認ワークフローツールです。

## 主要機能

### ユーザー管理
- **役割ベース認証**: 申請者、確認者、承認者、管理者の4つの役割
- **部署・役職管理**: ユーザーごとの部署と役職情報

### 申請管理
- **申請作成・編集**: タイトル、内容、種別、優先度、金額など詳細な申請情報
- **申請種別**: 経費申請、休暇申請、購入申請、その他
- **ステータス管理**: 下書き、提出済み、確認中、承認済み、却下、キャンセル
- **添付ファイル**: 複数ファイルの添付対応

### 承認ワークフロー
- **柔軟なフロー設定**: 申請種別ごとの承認フロー定義
- **2段階承認**: 確認→承認の段階的処理
- **条件分岐**: 金額や申請内容による自動フロー選択
- **コメント機能**: 各段階でのコメント・理由記録

### ダッシュボード
- **統計情報**: 申請数、承認待ち件数などの可視化
- **通知機能**: 承認待ち案件の一覧表示
- **検索・フィルタ**: ステータス、種別、申請者での絞り込み

## 技術仕様

- **言語**: PHP 8.1+
- **フレームワーク**: Laravel 10+
- **データベース**: MariaDB/MySQL
- **フロントエンド**: Laravel Blade + Bootstrap 5
- **認証**: Laravel標準認証

## データベース設計

### users（ユーザー）
- 基本情報: name, email, password
- 役割: role (applicant/reviewer/approver/admin)
- 組織情報: department, position

### applications（申請）
- 申請情報: title, description, type, status
- 金額・日程: amount, requested_date, due_date
- 関係: applicant_id → users

### approval_flows（承認フロー）
- フロー設定: name, application_type, step_count
- 条件設定: conditions (JSON)
- ステップ設定: flow_config (JSON)

### approvals（承認履歴）
- 承認情報: application_id, approver_id, step_number
- 種別: step_type (review/approve)
- 結果: status, comment, acted_at

## セットアップ手順

### Dockerを使用する場合（推奨）

#### 1. 環境要件
- Docker
- Docker Compose
- Python 3.x（テスト実行用）

#### 2. NewRelic版Docker環境のセットアップとテスト実行

##### 完全なビルドからテストまでの一括実行
```bash
# リポジトリクローン
git clone <repository-url>
cd nrkk-workflow

# NewRelic版Dockerのビルド・起動・テスト実行を一括で行う
docker-compose -f docker-compose.newrelic.yml build && \
docker-compose -f docker-compose.newrelic.yml down && \
docker-compose -f docker-compose.newrelic.yml up -d && \
test_env/bin/python tests/test_create_applications.py && \
test_env/bin/python tests/test_approve_applications.py
```

##### 個別実行する場合
```bash
# 1. Dockerイメージのビルド
docker-compose -f docker-compose.newrelic.yml build

# 2. 既存コンテナの停止・削除
docker-compose -f docker-compose.newrelic.yml down

# 3. コンテナの起動（バックグラウンド実行）
docker-compose -f docker-compose.newrelic.yml up -d

# 4. 申請作成テストの実行
test_env/bin/python tests/test_create_applications.py

# 5. 承認処理テストの実行
test_env/bin/python tests/test_approve_applications.py
```

#### 3. アクセス
- **アプリケーション**: http://localhost:8080
- **phpMyAdmin**: http://localhost:8081

#### 4. テスト環境のセットアップ（初回のみ）
```bash
# Python仮想環境の作成
python3 -m venv test_env

# 仮想環境のアクティベート
source test_env/bin/activate  # macOS/Linux
# または
test_env\Scripts\activate  # Windows

# 必要なパッケージのインストール
pip install requests python-dotenv
```

#### 5. その他のDocker操作コマンド
```bash
# ログ確認
docker-compose -f docker-compose.newrelic.yml logs -f

# 完全リセット（データベース含む）
docker-compose -f docker-compose.newrelic.yml down -v

# コンテナ内でコマンド実行
docker-compose -f docker-compose.newrelic.yml exec app php artisan migrate
docker-compose -f docker-compose.newrelic.yml exec app php artisan tinker

# コンテナの状態確認
docker-compose -f docker-compose.newrelic.yml ps
```

### ローカル環境でのセットアップ

#### 1. 環境要件
- PHP 8.1以上
- Composer
- MariaDB/MySQL 5.7以上
- Node.js (任意 - アセット管理用)

#### 2. インストール
```bash
# リポジトリクローン
git clone <repository-url>
cd nrkk-workflow

# 依存関係インストール
composer install

# 環境設定
cp .env.example .env
php artisan key:generate
```

#### 3. データベース設定
```bash
# .envファイルでデータベース設定
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=approval_workflow
DB_USERNAME=root
DB_PASSWORD=

# データベース作成
mysql -u root -p -e "CREATE DATABASE approval_workflow;"

# マイグレーション実行
php artisan migrate

# テストデータ投入
php artisan db:seed
```

#### 4. サーバー起動
```bash
php artisan serve
```

ブラウザで `http://localhost:8000` にアクセス

## テストユーザー

### 管理者
| 役割 | 名前 | メール | パスワード |
|------|------|--------|------------|
| 管理者 | 管理者 | admin@wf.nrkk.technology | password |

### 申請者（サンプル）
| 名前 | メール | パスワード | 組織 |
|------|--------|------------|------|
| 星野和子 | hoshino.kazuko@wf.nrkk.technology | password | テックイノベーション |
| 笹田純子 | sasada.junko@wf.nrkk.technology | password | テックイノベーション |
| 斉藤和明 | saito.kazuaki@wf.nrkk.technology | password | グリーンエネルギー |
| 青木翔太 | aoki.shota@wf.nrkk.technology | password | やまと建設 |
| 石川由紀 | ishikawa.yuki@wf.nrkk.technology | password | みどり食品工業 |

### 承認者（サンプル）
| 名前 | メール | パスワード | 組織 |
|------|--------|------------|------|
| 中村恵子 | nakamura.keiko@wf.nrkk.technology | password | テックイノベーション |
| 木村智子 | kimura.tomoko@wf.nrkk.technology | password | グリーンエネルギー |
| 佐藤太郎 | sato.taro@wf.nrkk.technology | password | みどり食品工業 |
| 鈴木花子 | suzuki.hanako@wf.nrkk.technology | password | さくら運輸 |

> 💡 **注意**: 全ユーザーのパスワードは `password` です。本番環境では必ず変更してください。

## 基本的な使用フロー

1. **申請者**が新規申請を作成
2. 申請内容に応じて**承認フロー**が自動選択
3. **確認者**が申請内容を確認
4. **承認者**が最終承認
5. 申請者に結果通知

## 申請承認の詳細フロー

### 一般申請フロー
1. 確認（reviewer）
2. 承認（approver）

### 高額経費申請フロー（10万円以上）
1. 確認（reviewer）
2. 承認（approver）
3. 最終承認（admin）

### 休暇申請フロー
1. 承認（approver）

## API拡張

将来的にREST APIとして拡張可能な設計となっています。

```php
// 例: API Routes
Route::apiResource('applications', ApplicationController::class);
Route::post('applications/{application}/submit', [ApplicationController::class, 'submit']);
```

## カスタマイズポイント

### 承認フローの追加
`ApprovalFlow`モデルで新しいフローを定義

### 申請種別の追加
マイグレーションで`applications.type`のenumを拡張

### 通知機能の実装
Laravel Notificationを使用した通知機能追加可能

## セキュリティ

- CSRF保護
- SQLインジェクション対策
- XSS対策
- 認可ポリシー（Policy）による権限制御

## トラブルシューティング

### よくある問題

**Q: マイグレーションエラー**
```bash
php artisan migrate:fresh --seed
```

**Q: 権限エラー**
```bash
sudo chown -R $USER:www-data storage/
sudo chmod -R 775 storage/
```

**Q: キャッシュクリア**
```bash
php artisan cache:clear
php artisan config:clear
php artisan view:clear
```

## ライセンス

MIT License

## サポート

Issue報告やプルリクエストをお待ちしています。
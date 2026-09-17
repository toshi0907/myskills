---
name: android-app-dev-foundations
description: >
  Kotlin + Jetpack Compose + GradleでAndroidアプリを開発する際に繰り返し踏みがちな
  技術的な落とし穴をまとめた汎用Skill。Kotlin 2.0以降でComposeを使うためのCompiler
  Gradleプラグイン必須化、RoomのDBスキーマ変更時のMigration運用、WorkManagerで
  15分未満の間隔の定期実行が必要な場合の自己連鎖パターン、複数コルーチンから並行に
  呼ばれる登録/解除処理の直列化とキャンセル安全性、位置情報(Geofencing/継続的な
  位置追跡)を使う場合の精度・権限・バックグラウンド制約まわりの注意点を扱う。
  Android(Kotlin/Compose)アプリの新規開発・機能追加で、Gradle/Composeプラグイン
  構成、DBスキーマ変更、定期実行処理、非同期処理の排他制御、位置情報機能の実装を
  行う際に確認すること。開発体制(スマホ単体+Claude Code特有のCI依存フロー等)は
  `android-mobile-only-dev`を参照する。
---

# Androidアプリ開発の技術基盤

## 目的

Kotlin + Jetpack Compose + Gradle構成のAndroidアプリでは、プロジェクトが変わっても
同じ種類の技術的な落とし穴を繰り返し踏みやすい。このSkillは、そうした汎用的な
知見を集約し、実装に着手する前に確認することで同じ問題を再発させないようにする。

## Gradle/Kotlin/Composeのプラグイン要件

- Kotlin 2.0以降でJetpack Composeを使うには、`org.jetbrains.kotlin.plugin.compose`
  というGradleプラグインの適用が別途必須(ルートの`build.gradle.kts`とアプリ
  モジュールの`build.gradle.kts`の両方)。従来の
  `composeOptions.kotlinCompilerExtensionVersion`による指定はKotlin 2.0以降は
  機能せず、適用漏れは「Compose Compiler Gradle plugin is required」という
  ビルドエラーになる。
- Kotlinのバージョンを上げる際は、この2箇所のプラグインバージョン指定を
  同期させる(片方だけ上げるとバージョン不整合でビルドが失敗する)。

## Room(SQLite)のスキーママイグレーション運用

- エンティティのスキーマを変更する際は、`@Database`の`version`を上げるのと
  同時に、必ずそのバージョン間の明示的な`Migration`を追加して`addMigrations()`に
  登録する。追加を怠ると、そのバージョン間で更新した既存ユーザーの端末内データが
  失われる。
- `fallbackToDestructiveMigration()`は、対応するMigrationが存在しない古い
  バージョンからの更新時のみ働く最終手段として残し、通常のスキーマ変更では
  使わない(実運用でのデータ消失に直結するため)。

## WorkManagerで15分未満の間隔の定期実行が必要な場合

- `PeriodicWorkRequest`は最短でも15分間隔までしか指定できない。より短い間隔で
  定期実行したい場合は、`OneTimeWorkRequest`が実行完了のたびに次回分を
  `ExistingWorkPolicy.REPLACE`で自己予約する「自己連鎖」パターンを使う。
- 処理対象が無くなった場合は次回分を予約せずに終えることで連鎖を自然に停止できる
  (ただし停止までに最大で実行間隔ぶんの遅延がある)。
- 途中で例外が発生すると連鎖が途切れて二度と実行されなくなるため、
  `doWork()`は次回分の予約を`try`/`finally`で囲み、処理内容の成否に関わらず
  必ず次回分を予約するようにする。

## 並行呼び出しの直列化とキャンセル安全性

- 複数の独立したコルーチン(ViewModelの操作・BroadcastReceiver・設定変更等)から
  並行に呼ばれうる登録/解除処理(通知のスケジューリング、ジオフェンスの登録、
  フォアグラウンドサービスの起動/停止等)は、単一の`Mutex`で直列化しないと
  呼び出しの順序が入れ替わり、新しい登録を古い解除呼び出しが後から取り消して
  しまう、といった不整合が起こりうる。
- Play Services等が返す非同期`Task`を`suspendCancellableCoroutine`で待つ処理、
  および一連の後始末処理(旧リソースの解除→新リソースの登録)は、呼び出し元の
  コルーチンスコープが途中でキャンセルされても中断されてほしくないことが多い。
  `withContext(NonCancellable)`で包み、ロックを保持したまま処理が尻切れに
  ならないようにする。
- `Job`の完了待ちには`CompletableDeferred`+`invokeOnCompletion`より`Job.join()`
  を使う方が安全な場合がある。`join()`は対象のJobが呼び出し前に既に終了して
  いても即座に戻るのに対し、`invokeOnCompletion`を後から登録する方式は、
  登録前にJobが終了済みだと二度と呼ばれず待ち合わせが永久に完了しないことがある。

## 位置情報(Geofencing/継続的な位置追跡)を使う場合の注意点

- **精度による誤検知**: GPS/ネットワーク測位の精度が低いタイミングでは、実際には
  動いていなくてもジオフェンスの内外判定が一瞬だけ反転し、ENTER/EXITが誤って
  発火することがある(特に半径に対して精度が粗い場合や、複数のジオフェンスの円が
  近接・重複している場合に起きやすい)。イベントを受け取っても即座に通知/確定
  処理をせず、一定時間(例: 3分)後続の反対方向イベントが来なければ確定する
  デバウンス処理を挟む。デバウンスの実装にはWorkManagerの`enqueueUniqueWork`+
  `ExistingWorkPolicy.REPLACE`(ユニークワーク名をタスク/対象ごとに分ける)が
  使いやすい。
- **Geofencing APIの評価間隔の粗さ**: OS側の内部評価間隔が間引かれ、実際には
  数時間前に起きていた遷移がまとめて遅れて通知される「遅延キャッチアップ」が
  起こりうる。即時性が必要なら、`GeofencingRequest.Builder.setNotificationResponsiveness()`
  で応答性を上げるか、Geofencing APIに頼らずフォアグラウンドサービスで
  `FusedLocationProviderClient.requestLocationUpdates`により自前で定期的に
  現在地を取得し圏内/圏外を判定する「継続的追跡」方式を選択肢として用意する
  (バッテリー消費とのトレードオフがあるため、ユーザーが方式を選べるようにすると
  よい)。
- **権限**: ジオフェンス登録・バックグラウンドでの位置取得には`ACCESS_FINE_LOCATION`
  に加え、API 29(Q)以降は`ACCESS_BACKGROUND_LOCATION`も別途必要。この権限は
  「常に許可」を選ばないと付与されないため、UI上で不足時にアプリ設定画面へ
  誘導するバナー等を用意する。
- **フォアグラウンドサービスでの位置取得**: `foregroundServiceType="location"`
  で`FOREGROUND_SERVICE`・`FOREGROUND_SERVICE_LOCATION`権限が必要。権限が
  揃っていても、端末側の位置情報サービス自体(設定アプリの「位置情報」トグル)
  がオフだと、Android 14以降は`startForeground`が`SecurityException`を投げる。
  `LocationManagerCompat.isLocationEnabled()`で事前に確認し、オフなら
  `stopSelf()`するなど、権限確認だけでは検知できないこのケースを別途弾く。
- **端末再起動での再登録**: ジオフェンス登録・継続的追跡のためのサービス起動は
  端末再起動で失われるため、`RECEIVE_BOOT_COMPLETED`を受けて、位置情報を使う
  未完了の対象を全件再登録する処理が必要。
- **通知の分離**: 位置情報の通知は、期限日時等の他の通知と別の通知チャンネル・
  タグを使う。同じ通知ID・タグを使い回すと、片方の通知がもう片方を通知欄から
  上書き消去してしまうことがある。
- **バックプレッシャー**: フォアグラウンドサービスで位置更新のコールバックを
  受けるたびに評価処理(DB読み書き・ワーカー予約等)を毎回起動すると、評価が
  更新間隔より遅い場合に未処理の位置情報が溜まり続ける。`Channel(CONFLATED)`
  に`trySend()`で積み、単一のコンシューマーコルーチンが順に処理する構成にすると、
  最新の1件のみが保持され、かつ到着順どおりに処理される。

## 注意

- このSkillは技術的な落とし穴の知見集であり、「どこまで自動で進めるか」の
  合意形成は`github-dev-flow`、スマホ単体+Claude Code特有の開発フロー・
  配布体制は`android-mobile-only-dev`が扱う。Android開発の依頼ではこれらの
  Skillと併用する。
- 内容はプロジェクト固有の機能実装(通知文言・画面構成等の具体的な業務ロジック)
  ではなく、他のAndroidアプリ開発でも再利用可能な技術要素に絞っている。

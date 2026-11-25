# Super Bonjour (SB)

**iTunes Wi-Fi同期の復旧手法 — 技術仕様・実装ガイド**

[English](en/README.md) | 公開候補版 0.2 / 2026-11-25

## 概要

Windows版iTunesで、以前にペアリングしたiPhoneのプライベートWi-Fiアドレスが変わった後、Wi-Fi経由で検出・同期できなくなる現象を調査しました。対象端末の既存のペアリング記録を保持し、Bonjourで通知される現在のMACアドレスを、既存記録のHostIDと認証タグの照合で特定します。その後、対象のplistの `WiFiMACAddress` だけを更新することで、開発環境ではWi-Fi同期を復旧できました。

これはAppleの公式修正ではなく、個人環境で実証した回避策です。iTunesが検出できなくなる内部原因は未確定です。プライベートアドレス機能を無効にし続ける必要がないことを目標としています。

## 読む順番

| 資料 | 内容 |
|---|---|
| [SPECIFICATION.md](SPECIFICATION.md) | 機能、初期登録、書き換えと復元、安全条件 |
| [ALGORITHM.md](ALGORITHM.md) | Bonjour、authTagの計算、判定、疑似コード |
| [TEST_RESULTS.md](TEST_RESULTS.md) | 匿名化した実機結果と未検証事項 |
| [AI_PROMPT.md](AI_PROMPT.md) | AIに自分専用の実装を作成させる指示文 |

## 対象と限界

- 実証環境はWindows 10、iTunes、Bonjour、iPhone 12 Pro Max。既存の信頼済みペアリングとWi-Fi同期設定が必要です。
- 対象端末ごとに異なるHostIDが使われた環境で、4条件の識別と統合版v1.72の同期復旧を確認しました。自然発生する長期ローテーション、他OS・他機種・Apple Devicesアプリは未検証です。
- HostIDを複数端末が共有する環境では、この照合だけで端末を一意に識別できません。その場合は自動更新を行いません。
- authTagは候補の照合情報であり、端末の暗号学的な完全認証ではありません。所有・管理する端末と信頼できるLANでのみ使用してください。

## 利用方法

本リポジトリは文書のみです。BAT、EXE、DLL、実際のplist、バックアップや生ログは配布しません。AIに仕様書・アルゴリズム・実測結果・指示文を渡すと、利用者自身の環境向けに実装を設計できます。AI生成コードは確認し、最初は読み取り専用の診断から実行してください。秘密鍵やペアリング記録はAI・外部サイト・公開リポジトリに送らないでください。

## 関連する先行事例（2026-11-25確認）

- [netmuxd Issue #56](https://github.com/jkcoxson/netmuxd/issues/56)：LinuxでBonjour通知MACとペアリング記録の不一致を報告し、MAC更新による復旧を報告。提案された更新スクリプトはUSB接続を必要とする。
- [AirBattery PR #210](https://github.com/lihaoyun6/AirBattery/pull/210)：macOSで同様のMAC更新を実装。PRの説明では、ローテーション後の再更新は次回のUSB接続時に行う。
- [GeoPort](https://github.com/davesc63/GeoPort)：iOSの位置情報シミュレーションを目的とする別プロジェクト。

SBの成果は、既知のmobdev2照合方式とペアリング記録のMAC更新を組み合わせ、Windows版iTunesのWi-Fi同期を、既存ペアリング・ローテーション設定を維持したままUSB再接続なしで復旧した個人環境の実証です。世界初や先行事例の不存在は主張しません。

## 参考資料・クレジット

- [pymobiledevice3: Wi-Fi端末認識の技術資料](https://doronz88.github.io/pymobiledevice3/latest/internals/idevice-protocol-layers/#wi-fi-recognizing-a-device-on-the-network) — mobdev2のauthTag方式。方式自体の新規発見を主張するものではありません。
- [pymobiledevice3 ソースリポジトリ](https://github.com/doronz88/pymobiledevice3) — 実装参照元。コードを転載する場合は原ライセンスを確認してください。
- [Apple: プライベートWi-Fiアドレス](https://support.apple.com/ja-jp/102509) — Appleが説明する機能の仕様。
- [RFC 5869: HKDF](https://www.rfc-editor.org/rfc/rfc5869) — 鍵導出の規格。

公開者が文書のライセンスを決定するまでは、再配布許諾を付与したものではありません。Appleおよび各プロジェクトとは独立した調査です。

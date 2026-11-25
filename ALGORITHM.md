# Super Bonjour — アルゴリズム

[README](README.md) | [English](en/ALGORITHM.md)

## 1. Bonjourから取得する情報

Bonjour DNS-SDで `_apple-mobdev2._tcp.local.` をbrowseし、各サービスをresolveする。取得対象はサービスインスタンス名、インターフェース、SRVホスト名・ポート、A/AAAA、TXT `identifier` と `authTag` / `authTag#N`。MACは実測されたサービス名の先頭（例 `02:00:00:00:00:01@…`、架空値）から厳密に解析する。ARPによる代用やホスト名での断定はしない。無効なMAC、ゼロ、マルチキャストは除外する。SSIDはPCの設定値からiPhoneの接続先として推定しない。

元のWindows実装ではBonjourのDNSServiceBrowse、DNSServiceResolve、DNSServiceQueryRecordを使用。実証した統合版の待ち時間はbrowse 15,000 ms、resolve 10,000 ms、A/AAAA 2,600 ms。待ち時間は設定可能にし、取得漏れがあっても未取得を成功と扱わない。古いOSの端末はTXTに照合情報がないことがある。

## 2. ペアリング記録

Lockdown内の端末用plistからHostIDを文字列として取得する。SystemConfiguration.plist等を除外。実装上の端末UDIDファイル名の対象形式は40桁の16進数、または8桁-16桁の16進数。未対応形式は登録前に確認する。XMLはDTDの外部参照を禁止し、必要ならバイナリplist専用パーサを利用。HostIDにGUID形式を強制しない。実測では数字だけの文字列等が使用されていた。

## 3. 認証タグの計算

pymobiledevice3のmobdev2方式を参考に、ローカルで以下を計算する。秘密鍵やEscrowBagは計算に不要。HostID・identifierをUTF-8の元文字列として扱い、大小文字変換、trim、GUIDのバイト列化等をしない。JSON等のエスケープではなく、デコード済みの文字列を用いる。

```text
IKM = UTF8(HostID)
PRK = HMAC-SHA512(key = 64 zero bytes, message = IKM)
OKM = HMAC-SHA512(key = PRK, message = byte 0x01)[first 32 bytes]
expected = HMAC-SHA256(key = OKM, message = UTF8(identifier))[first 8 bytes]
received = strict_Base64_decode(TXT authTag value)
match = length(received) == 8 AND constant_time_equal(expected, received)
```

上記はHKDF-SHA512（salt未指定、info空、出力32 B）を展開した疑似コード。各 `authTag` と `authTag#N` に対して比較する。TXTキーの重複、欠損、形式異常、resolve矛盾は取得不可として扱う。タグは候補を絞るための情報であり、MACやサービス名を暗号学的に認証するものではない。悪意のあるLAN・リプレイ等への防御としての完全な認証ではない。

### 合成テストベクトル

以下は実機と無関係な架空入力。UTF-8の文字列とBase64処理の自己テストに使う。

- HostID: `SB-DEMO-HOST-001`
- identifier: `11111111-2222-4333-8444-555555555555`
- expected tag (hex): `D0C98FD99271C4C3`
- expected tag (Base64): `0MmP2ZJxxMM=`

## 4. 照合判定

```text
records = load_device_records_read_only()
require target_UDID exists exactly once
require all relevant HostIDs readable
require target HostID is not shared with any other record

candidates = []
for service in browse_and_resolve():
    if service has no usable identifier/tags:
        log unavailable or no-auth; continue
    matches = all records whose computed tag matches any valid received tag
    if target is among multiple matches: stop without writing
    if matches == [target]:
        require valid service MAC and consistent interface responses
        candidates.add(service MAC and service evidence)
require exactly one distinct, consistent target service
return its MAC
```

同じサービスの複数インターフェース応答は重複整理する。対象らしい応答内の矛盾は中止。他の端末の情報取得失敗だけで、対象の一意な照合を無効にする必要はない。異なる端末から対象HostIDと同じタグが得られるなど複数候補がある場合は更新しない。未観測端末や共通HostIDがある場合の一意性は保証できないため、共有HostIDがある環境は診断専用とする。別途、信頼済みセッションによるUDID検証が必要であり、本プロジェクトでは未実装・未検証。

## 5. 更新フロー

```text
read live target plist and hash its bytes
find target MAC using fresh Bonjour responses
if same MAC: finish without writing
show old/new values; request explicit confirmation
close iTunes; elevate only if needed; lock against concurrent writes
re-read target; require original hash unchanged
re-run target discovery; require same MAC (UUID may legitimately change)
stop relevant Apple processes/services safely
re-read target under write lock; require unchanged
create and verify before-backup outside Lockdown
replace only the validated WiFiMACAddress field bytes
verify saved bytes and non-MAC byte identity; create verified after-backup
on write/verify failure: restore verified before-backup and verify again
restart previously stopped service; verify file again
ask before PC reboot; verify actual Wi-Fi sync in iTunes
```

参照: [mobdev2の方式と限界](https://doronz88.github.io/pymobiledevice3/latest/internals/idevice-protocol-layers/#wi-fi-recognizing-a-device-on-the-network)、[RFC 5869](https://www.rfc-editor.org/rfc/rfc5869)。

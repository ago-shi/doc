# ✅ Kali LinuxでLANG1 / LANG2をIME切替に使う手順まとめ
## （Keyball – Vial / Fcitx5 使用）

以下の目的を達成する手順です:
- LANG1 → 日本語入力（Anthy）
- LANG2 → 英数入力（Keyboard - Japanese(JIS)）
- Windows での動作と同じ感覚で切り替え

## ① Fcitx5 を日本語入力環境として設定する
### 1. 必要なパッケージをインストール
```
sudo apt install fcitx5 fcitx5-anthy fcitx5-config-qt fcitx5-module-keyboard
```

### 2. Fcitx5 の環境変数を設定

以下の内容を作成：
```
~/.xprofile

## 以下を追記：
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

ログアウト → 再ログイン

## ② Fcitx5 の入力メソッド順序を設定
### 1. Fcitx5 の入力メソッド一覧を確認
```
fcitx5-configtool
```

現在の入力メソッドが以下であることを確認：
- Anthy
- Keyboard - 日本語（JIS）

### 2. 順番を「Anthy → Keyboard-日本語」にする

※ Anhty が先、Keyboard が後ろ。

## ③ Keyball の LANG1/LANG2 を IME 切替に割り当てる

今回は GUI で設定できたとのことなので、その手順を明確に記述します。

### 1. Fcitx5 設定を開く
```
fcitx5-configtool
```

### 2. 左側メニューから「グローバルオプション」を選択

（Kali 2025.4 では「全般」ではなく「グローバルオプション」）

### 3. 「ホットキー」を開く

→ この中に以下の項目があります：
- 入力メソッド有効化のホットキー
- 入力メソッド無効化のホットキー

この 2 つが「IME の切替」に相当します。

### 4. LANG1 キー（日本語切替）を設定
手順
1. 「入力メソッド有効化のホットキー」をクリック
2. ダイアログが出たら Keyball の LANG1 ボタン（F13 or Hangul） を押す
3. 表示されたら確定

### 5. LANG2 キー（英数切替）を設定
手順
1. 「入力メソッド無効化のホットキー」をクリック
2. Keyball の LANG2 ボタン（F14 or Hangul_Hanja） を押す
3. 表示されたら確定

※仕組みの説明  
有効化 = Anthy（日本語）に切り替える  
無効化 = Keyboard-Japanese に戻る（英数）  
という Fcitx5 の動作を利用した方式です。  

## ④ 動作確認
現在の入力メソッドを確認
```
fcitx5-remote -n
## 期待動作
LANG1 → anthy
LANG2 → keyboard-jp
```

文字入力テスト：
- LANG1 → 日本語変換できるか
- LANG2 → 英数（JIS 配列のまま）になるか

## ⑤（任意）xbindkeys が不要なら停止

GUI で割り当てができたため、xbindkeys を使っている場合は停止しておく方が安全です。
```
killall xbindkeys

## 自動起動しているなら無効化：
chmod -x ~/.xbindkeysrc
```

✅ この設定で得られるメリット
- Windows と 完全に同じ操作感で LANG1/LANG2 で日本語/英数切替
- キーボード配列は 常に JIS 固定
- RDP 経由だと keycode が変わる問題も直接接続時と区別可能
- 設定ファイルを汚さず GUI のみで完結
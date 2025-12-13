# 📝 完全手順：LANG1 / LANG2 を Fcitx5 で統一

## 1. Fcitx5 インストール＆設定確認

Kali Linux で Fcitx5 をインストール
```
sudo apt install fcitx5 fcitx5-anthy
```

IME が Anthy（日本語）と keyboard-jp（英数字）の 2つあることを確認  
プロファイル順： Anthy → keyboard-jp

環境変数確認（X11）
```
echo $GTK_IM_MODULE   # fcitx
echo $QT_IM_MODULE    # fcitx
echo $XMODIFIERS      # @im=fcitx
echo $XDG_SESSION_TYPE # x11
```
## 2. RDP/ローカル共通のキーコード統一（F13/F14）

~/.Xmodmap を作成：
```
vi ~/.Xmodmap

## 内容：
keycode 121 = F14  # LANG2
keycode 122 = F13  # LANG1

## 即時反映：
xmodmap ~/.Xmodmap

## 確認：
xmodmap -pke | grep -E "121|122"
## 出力例:
## keycode 121 = F14 NoSymbol F14
## keycode 122 = F13 NoSymbol F13
```

## 3. ログイン時に自動適用

~/.xsessionrc を編集 / 作成：
```
vi ~/.xsessionrc

## 内容：
if [ -f "$HOME/.Xmodmap" ]; then
    xmodmap "$HOME/.Xmodmap"
fi
```

XFCE / RDP セッションで自動適用される

## 4. Fcitx5 側ショートカット設定

1. fcitx5-configtool を開く → グローバルオプション（Global Options）
2. 日本語入力（Anthy）切替：F13
3. 英数字入力（keyboard-jp）切替：F14

これで LANG1/LANG2 ボタンを押すと、日本語/英数字が切り替わる

## 5. 動作確認

直接接続でも RDP 接続でも同じ操作で切り替わるかテスト  
ターミナルや Mousepad、Firefox などで実際に LANG1 / LANG2 を押す

### ✅ ポイントまとめ
- LANG1/LANG2 を F13/F14 に統一することで、接続環境に関わらず同じ動作
- ~/.Xmodmap + ~/.xsessionrc で自動反映
- Fcitx5 のショートカットは Global Options で F13/F14 に登録

これで Windows / Kali 両方で同じボタンで日本語/英数字を切替可能
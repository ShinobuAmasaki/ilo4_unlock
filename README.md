This article README_ja.md is translated into Japanese by Shinobu Amasaki, and is originally from [kendallgoto/ilo4_unlock/README.md](https://github.com/kendallgoto/ilo4_unlock/blob/main/README.md) in English by Kendall Goto.

このファイルREADME_ja.mdは雨崎しのぶによって日本語に翻訳されたものであり、オリジナルはKendall Gotoにより英語で書かれた[kendallgoto/ilo4_unlock/README.md](https://github.com/kendallgoto/ilo4_unlock/blob/main/README.md)である。
この記事に書かれたことを実行して生じた、あらゆる損害に対して翻訳者は責任を負わない。


# ilo4\_unlock (Silence of the Fans)

以前はアクセスできなかったユーティリティにアクセスできる、HPEのiLO4ファームウェアにパッチを適用するためのツールキット。

具体的には、ファームウェアには、システムヘルス（`h`）、ファン調整（`fan`）、オンボード温度センサー（`ocsd`）、およびオプションチップヘルスシステムに関する新しいコマンドにSSH経由でアクセスできるようにファームウェアにパッチを当てた。この修正ファームウェアは/r/homelabユーザー向けに設計されており、管理者に対してiLO4搭載サーバーのHPのアグレッシブなファン・カーブを調整する機能を提供する（例えばDL380pやDL380p Gen 8, Gen 9などである）。もう一つの一般的な使用例は、HPE認定以外のPCI-eカードがシステムに使用されている場合に、サーバーのファンが最大になるのを防ぐことである。

**注意：現時点では、v2.77がパッチを適用できる最新のiLOである。このバージョン以降、HPは、v2.78およびv2.79へのパッチを有効にするコントロールユーティリティの多くを削除している。将来的には変更される可能性があるが、v2.79（現在の最新版）に便利なツールを導入するには、非常に多くの作業が必要になる。ここではパッチは正常に機能するが、便利な機能にアクセスできないだけである。**

## Legal

> There is risk for potential damage to your system when utilizing this code.
> If an error occurs during flashing, or you end up flashing corrupted firmware, the iLO will not be able to recover itself.
> The iLO's flash chip cannot be programmed on-board, and must be fully desoldered and reprogrammed to recover the functionally. 
> Additionally, utilizing the included new features may cause your server to overheat or otherwise suffer damage.
> Do not proceed with installing this firmware if you don't know what you're doing.
> **You have been warned.**
> There is no warranty for this code nor will I be responsible for any damage you cause. 
> I have personally only tested this firmware on my DL380p Gen8, and DL380e Gen8.

> This repo does not contain any iLO 4 binaries; unmodified or patched as they are owned by HP.
> Websites have, in the past, been served with cease and desist orders from HP for hosting iLO binaries.
> For security purposes, I encourage you to follow the steps listed to build the patched version of the iLO yourself, while verifying the contents of the patched code.

これらのコードを使用した場合、あなたのシステムに重大な損害を及ぼすリスクがある。
ファームウェアの書き込み時にエラーが生じた場合、または破損したファームウェアをフラッシュすることになった場合に、iLOはそれ自体を回復できなくなる。
iLOのフラッシュチップはオンボードでプログラムすることができないため、機能を回復するには完全にハンダを除去して再プログラムする必要がある。
さらに、含まれている新機能を利用すると、サーバーが過熱したり、損傷をうける可能性がある。
あなたが何をしようとしているか分からない場合は、このファームウェアのインストールを実行してはならない。
**あなたは警告を受けた。**
このコードに保証はなく、あなたが引き起こした損害に対して私は責任を負わない。
私は個人的にこのファームウェアをDL380p Gen8およびDL380e Gen8でのみテストした。

このリポジトリにはiLO4バイナリは含まれていない。未修正またはパッチが適用されたものはHPによって所有されている。
過去にいくつかのウェブサイトがiLOバイナリを提供することに関して、HPから停止命令を受けていました。
セキュリティ上の理由から、パッチを適用したコードの内容を確認しながら、記載されている手順に従ってパッチを適用したバージョンのiLOを自分で構築することを推奨する。


## Getting Started

Python 2.7を必要とする。私(Kendall Goto)はすべてをCent OS 8上に構築した。他のOS/環境では要件が異なる場合がある。セットアップに余分な労力がかかる場合には、ドキュメント化するのでKendall Gotoまで知らせてほしい。

*pro tip!* これをすべてLive CD上で書き出すように実行している場合は、必ず最初にiLOセキュリティを無効にすべきである。無効にしないと、再起動する必要がある。詳細については"Flashing Firmware"を参照せよ。

ここに、私のUbuntu 21.10 Live CD用の私のセットアップを記す：

```bash
sudo apt-add-repository universe
sudo apt update
sudo apt-get install python2-minimal git curl
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py && sudo python2 get-pip.py
python2 -m pip install virtualenv
git clone --recurse-submodules https://github.com/kendallgoto/ilo4_unlock.git
cd ilo4_unlock
python2 -m virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Building Firmware

```
./build.sh init # download necessary HPE binaries
#./build.sh [patch-name] -- see patches/ folder for more info on each patch!

./build.sh 277
# generate iLO v2.77 patched firmware. The build setup creates a build/ folder where all the artifacts are stored.
# The final firmware location will be printed at the end of the script, if no errors are produced.
```

## Flashing Firmware

結果として得られるファームウェアは、ファームウェア名の下にあるビルドディレクトリにある（例：V2.73ビルドの場合は、`build/ilo4_273.bin.patched`）。ウェブインターフェースからではファームウェアを書き出すことができないため、次の手順でファームウェアを書き出すことを推奨する。

1. 生成されたファームウェアを書き出し用のUSBキーにコピーする。
2. サーバの電源を切り、iLO4セキュリティオーバーライドを有効にする（私の場合、これはボード上の最初のDIPスイッチを切り替えた）。
3. ベアメタルLinuxインストールからサーバーを起動する。Ubuntu LiveCDは正常に動作する。
   - ベアメタルシステムとは、非仮想化システム上で起動するという意味。
4. すべてのHPモジュールがアンロードされていることを確認する（`sudo modprobe -r hpilo`）。
5. USBキーを差し込み、ファームウェアの名前を`ilo4_250.bin`に変更し、`sudo ./flash_ilo4 --direct`を実行してサーバーにパッチを適用する。
6. 点滅している間、システムのプラグを抜き、すべてを壊したくなる衝動を抑える。
   - **大きな音が鳴る**。
   - 消去に2分、書き出しに1分かかった。
7. パッチを適用したあと、サーバーをシャットダウンして電源を切り、iLOセキュリティオーバーライドを無効にする。

"Getting Started"の手順に従って、ビルド後に行ったことは以下の通り。

```bash
sudo modprobe -r hpilo
mkdir -p flash
cp binaries/flash_ilo4 binaries/CP027911.xml flash/
cp build/ilo4_277.bin.patched flash/ilo4_250.bin
cd flash
sudo ./flash_ilo4 --direct  # wait until the fans spin down
sudo shutdown -h now        # remove power and disable the Security Override after shutting down!
```

## Use
```
FAN:
Usage:

  info [t|h|a|g|p]
                - display information about the fan controller
                  or individual information.
  g             - configure the 'global' section of the fan controller
  g smsc|start|stop|status
          start - start the iLO fan controller
          stop - stop the iLO fan controller
          smsc - configure the SMSC for manual control
       ro|rw|nc - set the RO, RW, NC (no_commit) options
    (blank)     - shows current status
  t             - configure the 'temperature' section of the fan controller
  t N on|off|adj|hyst|caut|crit|access|opts|set|unset
             on - enable temperature sensor
            off - disable temperature sensor
            adj - set ADJUSTMENT/OFFSET
      set/unset - set or clear a fixed simulated temp (also 'fan t set/unset' for show/clear all)
           hyst - set hysteresis for sensor
           caut - set CAUTION threshold
           crit - set CRITICAL threshold
         access - set ACCESS method for sensor (should be followed by 5 BYTES)
           opts - set the OPTION field
  h             - configure the 'tacHometers' section of the fan controller
  h N on|off|min|hyst|access
             on - enable sensor N
            off - disable sensor N
            min - set MINIMUM tach threshold
           hyst - set hysteresis
 grp ocsd|show  - show grouping parameters with OCSD impacts
  p             - configure the PWM configuration
  p N on|off|min|max|hyst|blow|pctramp|zero|feton|bon|boff|status|lock X|unlock|tickler|fix|fet|access
             on - enable (toggle) specified PWM
            off - disable (toggle) specified PWM
            min - set MINIMUM speed
            max - set MAXIMUM speed
           blow - set BLOWOUT speed
            pct - set the PERCETNAGE blowout bits
           ramp - set the RAMP register
           zero - set the force ZEROP bit on/off
          feton - set the FET 'for off' bit on/off
            bon - set BLOWOUT on
           boff - set BLOWOUT off
         status - set STATUS register
           lock - set LOCK speed and set LOCK bit
         unlock - clear the LOCK bit
        tickler - set TICKLER bit on/off - tickles fans even if FAN is stopped
  pid           - configure the PID algorithm
  pid N p|i|d|sp|imin|imax|lo|hi  - configure PID paramaters
                                  - * Use correct FORMAT for numbers!
             p - set the PROPORTIONAL gain
             i - set the INTEGRAL gain
             d - set the DERIVATIVE gain
            sp - set SETPOINT
          imin - set I windup MIN value
          imax - set I windup MAX value
            lo - set output LOW limit
            hi - set output HIGH lmit
 MISC
  rate X        - Change rate to X ms polling (default 3000)
  ramp          - Force a RAMP condition
  dump          - Dump all the fan registers in raw HEX format
  hyst h v1..vN - Perform a test hysteresis with supplied numbers
  desc <0>..<15> - try to decode then execute raw descriptor bytes (5 or 16)
  actn <0>..<15> - try to decode then execute raw action bytes (5 or 16)
  debug trace|t X|h X|a X|g X|p X|off|on
                - Set the fine control values for the fan FYI level
  DIMM          - DIMM-specific subcommand handler
  DRIVE         - Drive temperature subcommand handler
  MB            - Memory buffer subcommand handler
  PECI          - PECI subcommand handler
 AWAITING DOCUMENTAION
  ms  - multi-segment info
  a N  - algorithms - set parameters for multi-segment.
  w   - weighting
```

`script/`フォルダーを参照してほしい。あまり使用されない関数については、関連する資料を参照してほしい。私はそれらを使用していないので、文書化していない。

## Getting Involved

>  Want to get involved? Check out [here](https://github.com/kendallgoto/ilo4_unlock/blob/main/CONTRIBUTING.md)!

> There aren't really any hard rules here! If you have some additional  research, please make a PR / reach out to me and I'd love to chat about  it.
>
> If you'd like to chat about my research, need help getting something setup, etc., please reach out and I'm happy to help.
>
> For the sake of maintaining a dependable `main` and due to the vulnerability of generating iLO patches, please fully  document any patches in PRs so that I can adaquetly vet & test them. Submitted code should never include HP intellectual property -- i.e.,  no binaries!
>
> Please submit PRs against the `dev` branch.
>
> Kendall Goto ([k@kgo.to](mailto:k@kgo.to)) /u/iamkgoto

## Credits

> - Thanks to the work of [Airbus Security Lab](https://github.com/airbus-seclab/ilo4_toolbox); whose previous work exploring iLO 4 & 5 was instrumental in allowing the development of modified iLO firmware.

> - And to [/u/phoenixdev](https://www.reddit.com/user/phoenixdev), whose original work on iLO4 v2.60 and v2.73 allowed for fans to be controlled in the first place.
>    This repository utilizes modified code from the iLO4 Toolbox. The  toolkit invokes code directly from the iLO4 Toolbox, as well as includes modified versions of Airbus Security Lab's original patching code to  perform the necessary patches. It also utilizes code originally written  by [/u/phoenixdev](https://www.reddit.com/user/phoenixdev) that was reverse-engineered from their patched v2.73 iLO4 firmware.

> The full documentation on how this code base was derived is fully detailed [in the research/ folder](https://github.com/kendallgoto/ilo4_unlock/blob/main/research/readme.md).

## Relevant Reading & Prior Work

>  [2019-10-02 /u/phoenixdev's preliminary writeup](https://www.reddit.com/r/homelab/comments/dc7dbc/silence_of_the_fans_preliminary_success_with/)
>
>  [2019-10-15 /u/phoenixdev's first release for v2.60](https://www.reddit.com/r/homelab/comments/di3vrk/silence_of_the_fans_controlling_hp_server_fans/)
>
>  [2020-06-30 /u/phoenixdev's second release for v2.73](https://www.reddit.com/r/homelab/comments/di3vrk/silence_of_the_fans_controlling_hp_server_fans/)
>
>  [Airbus Security Lab's iLO4 Toolbox](https://github.com/airbus-seclab/ilo4_toolbox)
>
>  [Airbus Security Lab's "Subverting your server through its BMC: the HPE iLO4 case" (written version)](https://airbus-seclab.github.io/ilo/SSTIC2018-Article-subverting_your_server_through_its_bmc_the_hpe_ilo4_case-gazet_perigaud_czarny.pdf)
>
>  Airbus Security Lab's "Subverting your server through its BMC: the HPE iLO4 case" (presented version)](https://airbus-seclab.github.io/ilo/RECONBRX2018-Slides-Subverting_your_server_through_its_BMC_the_HPE_iLO4_case-perigaud-gazet-czarny.pdf)

## 翻訳者によるメモ
翻訳者は、ML350 Gen9のiLO4に対して、このコードをv2.77ファームウェアに適用して実行した。
おおむね正常に稼働しているがいくつかの問題が発見された。
1. `h`, `fan`, `ocsd`などのコマンドは実行できるが、結果が標準出力されない。 
2. `fan p N max VALUE`などの実行は、ファンの出力を制御できるので、たしかに実行されてはいるようだが、VALUEと制御値の対応が一致していない。
  - たとえば、`fan p 2 max 20`とするとiLOの管理ページで確認できる出力は23%、`fan p 2 max 20`とすると出力は7%になる。

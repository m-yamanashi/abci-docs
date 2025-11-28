# Lustre Client Encryptionによる暗号化

## Lustre Client Encryption（fscrypt）の概要

Lustre File Systemでは、[fscrypt](https://github.com/google/fscrypt)を使用することで、クライアント側で暗号化を行うLustre Client Encryptionを使用できます。Lustre Client Encryptionはデータの暗号化および復号を利用者が行い、暗号化されたデータをLustre File Systemに保存します。

利用者自身で作成した鍵ファイルを利用して暗号化を行う手順については、[Lustre Client Encryptionの利用方法](#usage-of-lustre-client-encryption)をご参照ください。

## Lustre Client Encryption（fscrypt）の利用方法 {#usage-of-lustre-client-encryption}
ここでは、利用者自身で鍵ファイルを作成し、[fscrypt](https://github.com/google/fscrypt)を利用してクライアント側での暗号化を行う手順を示します。

### 鍵ファイルと暗号化対象ディレクトリの作成
鍵ファイル（secret.key）を作成します。この鍵ファイルのサイズはちょうど32 バイトとなっており、尚且つ生のバイナリデータを格納している必要があります。また、この鍵ファイルは安全に保管し、暗号化対象のディレクトリ内には置かないようにしてください。
```
[username@hnode001 ~]$ head --bytes=32 /dev/urandom > secret.key
```
暗号化対象とする空ディレクトリを作成します。
```
[username@hnode001 ~]$ mkdir /groups/grpname/dir01/
```
!!! note
    fscryptは安全のため、既存のファイルを暗号化するようには設計されていません。このため、fscryptの暗号化は空のディレクトリに対してのみ動作します。暗号化を有効化した後のディレクトリへ保存されたデータは、すべて暗号化の対象となります。

上記手順により作成された鍵ファイルと暗号化対象ディレクトリのフルパスを、[お問い合わせ](../contact.md)ページを参照のうえ、<abci3-qa@abci.ai> までご連絡ください。ご連絡受領後、ABCIサポートによりディレクトリの暗号化を行います。暗号化の処理が完了後、再度ご連絡させて頂きます。

### 暗号化ディレクトリの確認
以下によりディレクトリが暗号化されていることの確認を行えます。こちらのディレクトリへ保存されたデータは、すべて暗号化の対象となります。
```
[username@hnode001 ~]$ fscrypt status /groups/grpname/dir01/
username/groups/grpname/dir01/ is encrypted with fscrypt.

Policy:   0308e5a50202e90690a6ecab878b8bb6
Options:  padding:32  contents:AES_256_XTS  filenames:AES_256_CTS  policy_version:2
Unlocked: Yes

Protected with 1 protector:
PROTECTOR         LINKED  DESCRIPTION
091fd9b10db03140  No      raw key protector "pr_username_YYYYMMDDhhmmsssss"
```

### 暗号化ディレクトリのロックとアンロック
暗号化済みのディレクトリは、以下の手順でロック、アンロックを行えます。以下の手順では暗号化ディレクトリをロックしています。この状態ではディレクトリ保持者を含め、ディレクトリへ保存されたデータを確認できません。
```
[username@hnode001 ~]$ cat /groups/grpname/dir01/hello.tx
hello
[username@hnode001 ~]$ 
[username@hnode001 ~]$ fscrypt lock /groups/grpname/dir01/
/groups/grpname/dir01/ is now locked.
[username@hnode001 ~]$
[username@hnode001 ~]$ cat /groups/grpname/dir01/hello.tx
cat: /groups/grpname/dir01/hello.tx: Required key not available
```

以下の手順では暗号化ディレクトリをアンロックしています。これにより、ディレクトリへ保存されたデータを確認できます。
fscrypt の暗号・復号化は、鍵をノードのカーネルに保持することで行われます。そのため、別のノードから暗号化ディレクトリにアクセスする場合は、事前に以下のアンロック処理が必要となります。
```
[username@hnode001 ~]$ cat /groups/grpname/dir01/hello.tx
cat: /groups/grpname/dir01/hello.tx: Required key not available
[username@hnode001 ~]$ 
[username@hnode001 ~]$ fscrypt unlock /groups/grpname/dir01/
/groups/grpname/dir01/ is now unlocked and ready for use.
[username@hnode001 ~]$
[username@hnode001 ~]$ cat /groups/grpname/dir01/hello.tx
hello
```


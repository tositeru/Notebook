# Docker

## 操作早見表

- インストール `docker pull mongo`
- インストールされているイメージの確認 `docker image list`
- 公開されているイメージの検索 `docker search <keyward>`
- 実行 `docker run -name <image-name> mongo:<tag> -p <hostOS port>:<conatiner port>`
- デーモンとして実行 `docker run -name <image-name> -d mongo:<tag>`
- 実行中のコンテナ確認 `docker ps`
- 実行中のコンテナ確認 `docker stats` or `docker stats --no-stream`
- コンテナの停止 `docker stop <container-name>`
- コンテナの再開 `docker restart <container-name>`
- 停止中のコンテナを取り除く `docker remove <container-name>`
- 実行中のコンテナ削除 `docker kill <container-name>`
- コンテナの設定の更新 `docker update <container-name>`
- 指定したコンテナにコマンド実行する `docker exec <container-name> <command> <args...>`
- 指定したコンテナのログを確認する `docker logs <container-name>`
- 指定したコンテナで起動しているプロセスを表示する `docker top <container-name>`

### 全てのコンテナ/イメージを削除する                
docker rm $(docker ps -a -q)

### Delete all images
docker rmi $(docker images -q)

## Dockerfileからイメージを作成

次のコマンドでカレントディレクトリにある`Dockerfile`からイメージを作成する。

`docker build -t <repository-name>:<tag> .`

## Data Volume

通常コンテナを削除するとその中のデータも一緒に消されるが、データボリュームとして保存したものは削除されない。

[Manage data in Docker](https://docs.docker.com/storage/)

データボリュームには二通りの方法がある。

- Volume Docker Areaという場所に保存する(Linuxだと/var/lib/docker/volumes)
- Bind mounts host OS上の好きなディレクトリに保存する
  
Linuxでは`tmpfs`というメモリ上に保存する方法もある。

### Volumes

ボリュームは`docker volume create`で明示的に作成するかコンテナ作成時に一緒に作成できる。

ボリュームはHost OS上に格納され、コンテナの中にボリュームがマウントされたときはボリュームがあるディレクトリもコンテナへマウントされる。
これは実際にマウントと似ている。

マウントされたボリュームはDockerによって管理され、Host OSから離れる。

ボリュームは複数のコンテナへ同時にマウントされることも出来る。

`docker volume prune .`で使用していないボリュームを取り除くことも出来る。

ボリュームをマウントするとき名前をつけるか無名のままでいくか選択できる。
無名ボリュームにはDockerは他とかぶらないランダムな名前をつけてくれる。

ボリュームも**ボリュームドライバー**の使用をサポートしている。

例)
- `docker volume create <volume-name>` 作成
- `docker volume ls` リストアップ
- `docker volume inspect <volume-name>` 詳しい情報の表示
- `docker volume rm <volume-name>` 削除

`docker inspect <container-name>`で表示される`Mounts`項目でも使用しているボリュームを確認できる。



#### 使い方

`-v,--volume`と`--mount`の二種類のオプションがある。

`--mount`はDocker 17.06からは追加されたもので単一のコンテナにボリュームを指定できる。

両者に機能的な違いはないので、**新しいユーザーには`--mount`のほうが単純なので使うことを勧めている。**

・使い方

- `--mount` `,`区切りの複数のKey/Valueから成る。KeyとValueの間は`=`を入れる。
    指定できるKeyには以下のものがある。
    1. `type` マウントのタイプ。`bind`、`volume`、`tmpfs`を指定できる。
    2. `source`|`src` マウントのソース。 ボリューム名を指定する。無名ボリュームの場合は省略できる。
    3. `destination`|`dst`|`target` コンテナ内でのマウント先となるファイル/ディレクトリパス
    4. `readonly` 任意のオプション。読み取り専用としてBind mountsを設定する。
    5. `volume-opt` 任意のオプション。複数あってもいい。Key/Valueのペアの形を取る。
    例) `docker run --mount 'type=volume,src=<volume-name>,dst=<container-path>,volume-driver=local'`
- `-v` `:`区切った3つのフィールドから成る。
  1. ボリューム名。無名ボリュームの場合は省略できる。
  2. マウント先となるコンテナ内のパス 
  3. 任意。`,`区切りのオプション。
  例)`docker run -v <volume-name>:<container-path>`

- `docker run -v <volume-name>:<> ...` ボリュームを指定してコンテナを実行
- `docker run -v <volume-name or id> --volume-driver <driver-name>` ボリュームのドライバーも指定してコンテナを実行。が、`--mount`を使うことを推奨してる。
- `docker run --volumes-from` 他のコンテナが使っているボリュームを使用してコンテナを実行


### Bind mounts

昔からある方法。

Bind mountを使うとHost OSのディレクトリやファイルはコンテナにマウントされる。
マウントされたファイルなどはHost OS上のフルパスによって参照される。

これらのファイルはDockerホストには存在してなくてもよく、実行中に作成することも出来る。

Bind mountはよいパフォーマンスを持つが、可能なディレクトリ構造はHost OSに依存してしまう。

新しいDockerアプリを作るときは名前付きのボリュームを使ってほしい。

Bind mountはDocker CLIコマンドで直接管理することはできない。

## Dockerfile 命令一覧

### COPY

[COPY](https://docs.docker.com/engine/reference/builder/#copy)

COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]

<src>から<dest>へファイル/ディレクトリをコピーする

<src>: コピーするファイル。
<dest>: 出力先のディレクトリ。WORKDIRからの相対パスまたは絶対パス

### WORKDIR

[WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir)

WORKDIR /path/to/workdir

RUNやCMD、ENTRYPOINT、COPY、ADD等の作業ディレクトリを指定する

### CMD

[CMD](https://docs.docker.com/engine/reference/builder/#cmd)

CMD ["executable","param1","param2"] (exec form, this is the preferred form)
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
CMD command param1 param2 (shell form)

## ERROR: error while removing network: network <network-name> id <network-id> has active endpointsと出た時の対処法

何らかの理由で<network-name>がDockerコンテナと接続していることが原因。

`docker network inspect <network-name>`

で<network-name>の接続情報を表示し、接続しているコンテナを探す。

情報はJSON形式で表示され、Containersキーに接続中のコンテナが記述されている。

確認できたら`docker stop <container-name>`で接続中のコンテナを停止させ、再度<network-name>を停止させると無事目的を達成できる。

```bash
docker network inspect <network-name>
docker stop <container-name>
docker network rm <network-name>
```

## 起動しているDockerコンテナ環境を確認したい時

`docker exec -it <container-id> bash`

CUIに慣れておくと作業がやりやすいと思う。

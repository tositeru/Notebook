# Docker

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

確認できたら`docker stop <container-name>`で接続中のコンテナを停止させ、再度<network-network>を停止させると無事目的を達成できる。

```bash
docker network inspect <network-name>
docker stop <container-name>
docker network rm <network-name>
```

## 起動しているDockerコンテナ環境を確認したい時

`docker exec -it <container-id> bash`

CUIに慣れておくと作業がやりやすいと思う。

# 全てのコンテナ/イメージを削除する                
docker rm $(docker ps -a -q)
# Delete all images
docker rmi $(docker images -q)

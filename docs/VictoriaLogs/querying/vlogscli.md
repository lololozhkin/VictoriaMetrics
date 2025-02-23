---
weight:
title: vlogscli
disableToc: true
menu:
  docs:
    parent: "victorialogs-querying"
    weight: 1
---

`vlogsqcli` is an interactive command-line tool for querying [VictoriaLogs](https://docs.victoriametrics.com/victorialogs/).
This tool can be obtained from the linked release pages at the [changelog](https://docs.victoriametrics.com/victorialogs/changelog/)
or from [docker images](https://hub.docker.com/r/victoriametrics/vlogscli/tags):

### Running `vlogscli` from release binary

```sh
curl -L -O https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v0.33.0-victorialogs/vlogscli-linux-amd64-v0.33.0-victorialogs.tar.gz
tar xzf vlogscli-linux-amd64-v0.33.0-victorialogs.tar.gz
./vlogscli-prod
```

### Running `vlogscli` from Docker image

```sh
docker run --rm -it docker.io/victoriametrics/vlogscli:v0.33.0-victorialogs
```

By default `vlogscli` sends queries to [`http://localhost:8429/select/logsql/query`](https://docs.victoriametrics.com/victorialogs/querying/#querying-logs).
The url to query can be changed via `-datasource.url` command-line flag. For example, the following command instructs
`vlogsql` sending queries to `https://victoria-logs.some-domain.com/select/logsql/query`:

```sh
./vlogsql -datasource.url='https://victoria-logs.some-domain.com/select/logsql/query'
```

If some HTTP request headers must be passed to the querying API, then set `-header` command-line flag.
For example, the following command starts `vlogsql`,
which queries `(AccountID=123, ProjectID=456)` [tenant](https://docs.victoriametrics.com/victorialogs/#multitenancy):

```sh
./vlogsql -header='AccountID: 123' -header='ProjectID: 456'
```

After the start `vlogsql` provides a prompt for writing [LogsQL](https://docs.victoriametrics.com/victorialogs/logsql/) queries.
The query can be multi-line. It is sent to VictoriaLogs as soon as it contains `;` at the end or if a blank line follows the query.
For example:

```sh
;> _time:1y | count();
executing [_time:1y | stats count(*) as "count(*)"]...
{
  "count(*)": "1923019991"
}
duration: 0.688s
```

Query execution can be interrupted at any time by pressing `Ctrl+C`.

Type `q`, `quit` or `exit` and then press `Enter` for exit from `vlogsql`.

If the query response exceeds vertical screen space, `vlogsql` pipes query response to `less` utility,
so you can scroll the response as needed. This allows executing queries, which potentially
may return billions of rows, without any problems at both VictoriaMetrics and `vlogsql` sides,
thanks to the way how `less` interacts with [`/select/logsql/query`](https://docs.victoriametrics.com/victorialogs/querying/#querying-logs):

- `less` reads the response when needed, e.g. when you scroll it down.
  `less` pauses reading the response when you stop scrolling. VictoriaLogs pauses processing the query
  when `less` stops reading the response, and automatically resumes processing the response
  when `less` continues reading it.
- `less` closes the response stream after exit from scroll mode (e.g. by typing `q`).
  VictoriaLogs stops query processing and frees up all the associated resources
  after the response stream is closed.

See also [`less` docs](https://man7.org/linux/man-pages/man1/less.1.html) and
[command-line integration docs for VictoriaMetrics](https://docs.victoriametrics.com/victorialogs/querying/#command-line).


## Query history

`vlogsql` supports query history - press `up` and `down` keys for navigating the history.
By default the history is stored in the `vlogsql-history` file at the directory where `vlogsql` runs,
so the history is available between `vlogsql` runs.
The path to the file can be changed via `-historyFile` command-line flag.

Quick tip: type some text and then press `Ctrl+R` for searching queries with the given text in the history.
Press `Ctrl+R` multiple times for searching other matching queries in the history.
Press `Enter` when the needed query is found in order to execute it.
Press `Ctrl+C` for exit from the `search history` mode.
See also [other available shortcuts](https://github.com/chzyer/readline/blob/f533ef1caae91a1fcc90875ff9a5a030f0237c6a/doc/shortcut.md).

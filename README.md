# gonx

`gonx` is access log parser for any format.

## Usage

The library provides `Reader` type and two constructors for it.

Common constructor `NewReader` gets opened file (any `io.Reader` in fact) and log format of type `string` as argumets.
[Format](#format) is in form os Nginx `log_format` string.

```go
reader := gonx.NewReader(file, format)
```

`NewNginxReader` provides more magic. It gets log file `io.Reader`, Nginx config file `io.Reader`
and `log_format` name `string` as a third. The actual format for `Parser` will be extracted from
given Nginx config.

```go
reader := gonx.NewNginxReader(file, NginxConfig, format_name)
```

`Reader` implements `io.Reader`. Here is example usage

```go
for {
	rec, err := reader.Read()
	if err == io.EOF {
		break
	}
	// Process the record... e.g.
}
```

See more examples in `example/*.go` sources.

## Performance

NOTE All benchmarks was made on my old *11" MacBook Air 2011*, so you should get the better results for your brand new hardware ;-)

I have a few benchmarks for parsing `string` log record into `Entry` using `gonx.Parser`

	BenchmarkParseSimpleLogRecord      100000            19457 ns/op
	BenchmarkParseLogRecord             20000            84425 ns/op

And here is some real wold stats. I got ~300Mb log file with ~700K records and process with [simple scripts](https://github.com/kitabisa/gonx/tree/master/benchmarks).

* Reading whole file line by line with `bufio.Scanner` without any other processing takes a *one second*.
* Read in the same manner plus parsing with `gonx.Parser` takes *about 80 seconds*
* But for reading this file with `gonx.Reader` which parses records using separate goroutines it takes *about 45 seconds* (but I want to make it faster)

## Format

As I said above this library is primary for Nginx access log parsing, but it can be configured to parse any
other format. `NewReader` accepts `format` argument, it will be transformed to regular expression and used
for log line by line parsing. Format is Nginx-like, here is example

	`$remote_addr [$time_local] "$request"`

It should contain variables in form `$name`. The regular expression will be created using this string
format representation

	`^(?P<remote_addr>[^ ]+) \[(?P<time_local>[^]]+)\] "(?P<request>[^"]+)"$`

`Reader.Read` returns a record of type `Entry` (which is customized `map[string][string]`). For this example
the returned record map will contain `remote_addr`, `time_local` and `request` keys filled with parsed values.

## Stability

This library API and internal representation can be changed at any moment, but I guarantee that backward
capability will be supported for the following public interfaces.

* `func NewReader(logFile io.Reader, format string) *Reader`
* `func NewNginxReader(logFile io.Reader, nginxConf io.Reader, formatName string) (reader *Reader, err error)`
* `func (r *Reader) Read() (record Entry, err error)`

## Changelog

All major changes will be noticed in [release notes](https://github.com/kitabisa/gonx/releases).

## Contributing

Fork the repo, create a feature branch then send me pull request. Feel free to create new issues or contact me using email.

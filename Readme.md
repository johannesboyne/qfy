# qfy

  Tiny command line program that accepts a template and outputs fake data.

  ![](https://cldup.com/RZoAhReDqN.gif)

## Examples

```bash
# publish email to nsq every 1ms.
echo '{"email":"{{email}}", "subject": "welcome!"}' \
  | qfy --tick 1ms \
  | json-to-nsq --topic users

# add users to FoundationDB.
echo "'set {{username}} {{avatar}}'" \
  | qfy \
  | xargs -L1 -n3 fdbcli --exec

# add users to MongoDB.
echo "'db.users.insert({ name: \"{{name}}\" })'" \
  | qfy \
  | xargs -L1 -n1 mongo --eval

# add users to Redis.
echo "set {{username}} {{avatar}}" \
  | qfy \
  | xargs -L1 -n3 redis-cli

# send a single request using curl.
echo 'country={{country}}' \
  | qfy --max 1 \
  | curl -d @- httpbin.org/post
```

## Installation

```bash
$ go get github.com/peteraba/qfy
```

## Usage

```text

Usage: qfy
  [--tick d]
  [--max n]
  [--batch n]
  [--list]
  [--concurrent]

  qfy -h | --help
  qfy -v | --version

Options:
  --tick d        generate data every d [default: 10ms]
  --max n         generate data up to n [default: -1]
  --batch n       batch size for concurrent runs [default: 100]
  --list d        list all available generators
  --concurrent    skip ticks and generate fake data concurrently
  -v, --version   show version information
  -h, --help      show help information

```

## Generators

```text
  avatar
  color
  country
  country.code
  domain
  domain.name
  domain.tld
  double
  email
  event.action
  http.method
  id
  ipv4
  ipv6
  latitude
  longitude
  mac.address
  name
  name.first
  name.last
  product.category
  product.name
  smartdouble:desiredStdDev,desiredMean,min,max
  smartdate:format,deviationDays,rangeDays
  smartunixtime:deviationDays,rangeDays
  state
  state.code
  timezone
  unixtime
  username
  objectId (MongoDB/BSON)
```

### Generator attributes

#### Smart double

All arguments are optional.

- Desired standard deviation
- Desired mean
- Desired minimum value
- Desired maximum value

More info: https://golang.org/pkg/math/rand/#Rand.NormFloat64

#### Smart date

- Date format. Allowed values:
  - ANSIC       ("Mon Jan _2 15:04:05 2006")
  - UnixDate    ("Mon Jan _2 15:04:05 MST 2006")
  - RubyDate    ("Mon Jan 02 15:04:05 -0700 2006")
  - RFC822      ("02 Jan 06 15:04 MST")
  - RFC822Z     ("02 Jan 06 15:04 -0700" // RFC822 with numeric zone)
  - RFC850      ("Monday, 02-Jan-06 15:04:05 MST")
  - RFC1123     ("Mon, 02 Jan 2006 15:04:05 MST")
  - RFC1123Z    ("Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone)
  - RFC3339     ("2006-01-02T15:04:05Z07:00")
  - RFC3339Nano ("2006-01-02T15:04:05.999999999Z07:00")
  - Kitchen     ("3:04PM")
  - Stamp       ("Jan _2 15:04:05")
  - StampMilli  ("Jan _2 15:04:05.000")
  - StampMicro  ("Jan _2 15:04:05.000000")
  - StampNano   ("Jan _2 15:04:05.000000000")
  - SqlDateTime ("2006-01-02 15:04:05")
  - SqlDate     ("2006-01-02")
  - SqlTime     ("15:04:05")
- Deviation in days
- Range in days

#### Smart unixtime

- Deviation in days
- Range in days

## Numbering

You can define numbers in your template to reference previously defined templates:

```
echo '{"email":"{{email}}", "email_repeated": "{{0}}"}' \
  | qfy --max 1
```

This will output something similar to the following:
```
{"email":"tomaslau16@example.us","email_repeated":"tomaslau16@example.us"}
```

## License

  (MIT), 2014 Amir Abu Shareb.

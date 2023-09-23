# Restreamer REST API

## Login to restreamer server
~~~
$ curl -X 'POST' \
  'https://teemie1-relay.duckdns.org:8181/api/login' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "username": "admin",
  "password": "*****"
}'
~~~
## Show process
~~~
$ curl -X 'GET' \
  'https://teemie1-relay.duckdns.org:8181/api/v3/process' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer ACCESS_TOKEN'
~~~
## Show probe process
~~~
$ curl -X 'GET' \
  'https://teemie1-relay.duckdns.org:8181/api/v3/process/restreamer-ui:ingest:a1bf161c-c0af-4cd3-ae7b-c35f72d53e00/probe' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer ACCESS_TOKEN'
~~~

## List active session
~~~
$ curl -X 'GET' \
  'https://teemie1-relay.duckdns.org:8181/api/v3/session/active' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer ACCESS_TOKEN'
~~~
* ACCESS_TOKEN from the previous login

## Show metrics
~~~
$ curl -X 'GET' \
  'https://teemie1-relay.duckdns.org:8181/api/v3/metrics' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer ACCESS_TOKEN'
~~~

## Session summary
~~~
$ curl -X 'GET' \
  'https://teemie1-relay.duckdns.org:8181/api/v3/session' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer ACCESS_TOKEN'
~~~

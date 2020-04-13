# ping_to_json

Calls `ping` and packages the results as JSON for piping to other bash scripts.

Allows you to easily ping a remote server from a script. I use this script, run by cron, to log the latency on international traffic leaving my home.

In my case, using the IP addresses listed in Amazon's *EC2 Reachability Test* [list](http://ec2-reachability.amazonaws.com/), I ping `us_east_1` (Virginia), and `eu_west_1` (Dublin):

```
ruby ping.rb 3.80.0.0 | curl -H "Content-Type: application/json" -X POST -d "$(</dev/stdin)" http://<target>/?source=ping_us_east_1
ruby ping.rb 3.248.0.0 | curl -H "Content-Type: application/json" -X POST -d "$(</dev/stdin)" http://<target>/?source=ping_eu_west_1
```

Note, the `<target>` referenced is a local relay which forwards data to an Amazon API Gateway. See [here](https://github.com/renenw/relay) for an explantion of how (and why) I follow this approach.

# ping_to_json

Allows you to easily ping a remote server from a script.

The script calls `ping` and packages the results as JSON for piping to other bash scripts.

I use this script, run by cron on a Raspberry Pi, to log the latency on international traffic leaving my home. In my case, using the IP addresses listed in Amazon's *EC2 Reachability Test* [list](http://ec2-reachability.amazonaws.com/), I ping `us_east_1` (Virginia), and `eu_west_1` (Dublin):

```
ruby ping.rb 3.80.0.0 | curl -H "Content-Type: application/json" -X POST -d "$(</dev/stdin)" http://<target>/?source=ping_us_east_1
ruby ping.rb 3.248.0.0 | curl -H "Content-Type: application/json" -X POST -d "$(</dev/stdin)" http://<target>/?source=ping_eu_west_1
```

Note, the `<target>` referenced is a local relay which forwards data to an Amazon API Gateway. See [here](https://github.com/renenw/relay) for an explantion of how (and why) I follow this approach.

## Setup

Clone the repo, or grab ping.rb from the repo and save it locally.

Install ruby: `sudo apt-get install ruby`

Run it: `ruby ping.rb google.com`

## For completeness...

To monitor bandwidth (as opposed to latency), I use Ookla's `speedtest` against two key domestic servers: My ISP, and a the key domestic exchange (SAIX). I first tried to use speedtest (and speedtest-cli) to test internationally. But it there are two key problems with this approach:

1. Ookla clearly want you to test "locally". Their documentation suggests that their tests are optimised as such. And the server listings don't include, and the command line code, will not allow you to test against, far flung servers.
1. Ping is cheap - it can run all day. The Ookla tests are somewhat more onerous. Accordingly, I run the heavy tests once a day, and then ping every minute.

There's also a catch with cron: it doesn't pipe in the way you'd expect. As such, we need to run the lines usings `bash -c '<commands>'`.

My cron file is as follows:

```
  30  3    *   *   *   bash -c 'speedtest --accept-license --format=json --server-id 5921 | curl -H "Content-Type: application/json" -X POST -d "$(</dev/stdin)" http://<target>/?source=network_cybersmart'
  40  3    *   *   *   bash -c 'peedtest --accept-license --format=json --server-id 1879 | curl -H "Content-Type: application/json" -X POST -d "$(</dev/stdin)" http://<target>/?source=network_saix'

   *  *    *   *   *   bash -c 'ruby ping.rb  3.80.0.0 | curl -H "Content-Type: application/json" -X POST -d "$(</dev/stdin)" http://<target>/?source=ping_us_east_1'
   *  *    *   *   *   bash -c 'ruby ping.rb 3.248.0.0 | curl -H "Content-Type: application/json" -X POST -d "$(</dev/stdin)" http://<target>/?source=ping_eu_west_1'
```

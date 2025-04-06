# fw-fanctrl on Guix

The packages `ectool` and `fw-fanctrl` are available on the [panther](https://channels.pantherx.org/panther.git/plain/README.md) channel.

### Package

Put config in place:

```bash
guix shell fw-fanctrl
sudo mkdir /etc/fw-fanctrl
sudo cp $(dirname $(dirname $(readlink -f $(which fw-fanctrl))))/lib/python3.10/site-packages/fw_fanctrl/_resources/config.json /etc/fw-fanctrl/config.json
```

Run:

```bash
sudo fw-fanctrl run lazy
```

### Source

Here's how you can quickly test fw-fanctrl on guix:

```bash
guix shell python ectool
python3 -m venv venv
source venv/bin/activate
sudo fw-fanctrl run laziest --config src/fw_fanctrl/_resources/config.json
```

To reset to default:

```bash
sudo ectool autofanctrl
```

## Alternative

My previous Thinkpad X1 Gen9 (i7-8565U) runs a lot quieter than the Framework, but this comes down to two things:

- The CPU is usually throttled to ~ 2.7 / 4.6 Ghz
- The fan doesn't spin faster than ~ 3400 RPM

On the Framework, I can easily max out the ~ 4.9 Ghz, but the fan can reach 7000++ RPM.

### Disable Turbo Boost

Disabling turbo boost can make a huge difference, especially when CPU usage spikes.

Check if turbo boost is enabled (1 = enabled):

```bash
$ cat /sys/devices/system/cpu/cpufreq/boost
1
```

Disable it:

```bash
$ echo 0 | sudo tee /sys/devices/system/cpu/cpufreq/boost
0
```

Enable it:

```bash
$ echo 1 | sudo tee /sys/devices/system/cpu/cpufreq/boost
1
```

You can achieve the same via system config:

```scheme
(service tlp-service-type
         (tlp-configuration
           (cpu-scaling-governor-on-ac (list "balanced" "performance"))
           (cpu-boost-on-ac? #f)
           (cpu-scaling-governor-on-bat (list "low-power"))
           (cpu-boost-on-bat? #f)
           (sched-powersave-on-bat? #t)))
```

### CPU Frequency

Lowering the max-speed, can give you reasonable performance, in quiet environments, under heavy load:

```bash
sudo cpupower frequency-set -u 1Ghz
sudo cpupower frequency-set -u 2.4Ghz
```

and here's the governor, to apply to the frequency range:

```bash
sudo cpupower frequency-set --governor powersave
sudo cpupower frequency-set --governor performance
```
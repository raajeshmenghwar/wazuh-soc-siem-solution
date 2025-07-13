# Wazuh Manager Tuning Guide

This guide provides performance tuning, optimization, and reliability tips for the Wazuh Manager. It's especially useful when handling large-scale environments, reducing noise, and optimizing system resources.

---

## Why Tuning Matters

By default, Wazuh is configured for general use. In high-load or enterprise environments, tuning helps:

- Reduce false positives and unnecessary noise
- Improve detection performance and log handling
- Lower CPU/memory usage on the manager
- Speed up rule matching and syscheck scans

---

## Key Areas to Tune

### 1. Syscheck Frequency and Scope

Syscheck monitors files for changes, but scanning too many files too often consumes CPU and I/O.

#### Recommendations:

- Avoid scanning entire `/` or large, volatile directories like `/tmp`, `/var/log`, or `/proc`.
- Define precise directories in `ossec.conf` under the `<syscheck>` block:

```xml
<syscheck>
  <frequency>7200</frequency> <!-- Every 2 hours -->
  <directories check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
  <ignore>/etc/mtab</ignore>
  <scan_on_start>no</scan_on_start>
</syscheck>
````

| Setting         | Recommendation                               |
| --------------- | -------------------------------------------- |
| `frequency`     | Set between `3600–14400` seconds (1–4 hours) |
| `scan_on_start` | `no` (prevents long startup delays)          |
| `check_all`     | Only for critical dirs, not entire system    |

---

### 2. Rule Set Optimization

Too many active rules slow down the manager and increase alert noise.

#### Tips:

* Disable unused or irrelevant rule files by commenting them in `ossec.conf`:

```xml
<rules>
  <include>ruleset/rules/apache_rules.xml</include>
  <!-- <include>ruleset/rules/dhcpd_rules.xml</include> -->
</rules>
```

* Use only what's required for your use case (e.g., no Windows rules on a Linux-only environment).
* Move all custom rules to `local_rules.xml`.

---

### 3. Decoders and Event Noise Reduction

* Remove or disable decoders for services you don’t use.
* Reduce noisy log inputs (e.g., frequent CRON jobs, mail logs) using `<ignore>` blocks in `ossec.conf`.

---

### 4. System Resource Considerations

| Resource | Recommendation                                                     |
| -------- | ------------------------------------------------------------------ |
| **CPU**  | Minimum 4 vCPUs for medium deployments                             |
| **RAM**  | 8–16 GB for larger agent bases                                     |
| **Disk** | Use **SSD** for faster indexing and scanning                       |
| **I/O**  | Mount `/var/ossec/queue` on tmpfs (RAM) for performance (optional) |

#### Mount RAM Disk for temporary logs (optional):

```bash
sudo mount -t tmpfs -o size=512M tmpfs /var/ossec/queue
```

Add to `/etc/fstab` for persistence:

```
tmpfs /var/ossec/queue tmpfs defaults,size=512M 0 0
```

---

### 5. Reducing Active-Response Load

* Disable or limit `active-response` scripts that are overly aggressive or unused.
* Log instead of act where possible:

```xml
<active-response>
  <command>host-deny</command>
  <level>10</level>
  <timeout>600</timeout>
  <reconnect>yes</reconnect>
</active-response>
```

---

## Monitoring Wazuh Manager Performance

Use the following commands and logs to monitor load:

```bash
# View Wazuh Manager logs
sudo tail -f /var/ossec/logs/ossec.log

# Check system resource usage
htop

# Check syscheck performance
grep "syscheck" /var/ossec/logs/ossec.log
```

---

## Best Practices Summary

| Area           | Best Practice                          |
| -------------- | -------------------------------------- |
| **Syscheck**   | Limit directories, adjust frequency    |
| **Rules**      | Enable only needed rules               |
| **Decoders**   | Remove unused log sources              |
| **Hardware**   | Use SSDs and 4+ CPUs                   |
| **RAM Usage**  | Use tmpfs for queue/ if needed         |
| **Monitoring** | Regularly check logs and CPU/mem usage |

---

## Advanced Tuning (Optional)

* Use **multi-node clustering** if managing 1K+ agents
* Set **alerts per second** limits using `analysisd.threshold` (advanced use)
* Tune `agent-buffer` size in `ossec.conf` if events are getting lost

---

## Resources

* [Official Wazuh Performance Guide](https://documentation.wazuh.com/current/deployment-capabilities/performance/index.html)
* [Wazuh Tuning Tips](https://documentation.wazuh.com/current/deployment-capabilities/performance/tuning.html)

---

By applying these tuning techniques, you’ll improve stability, reduce resource consumption, and make your Wazuh Manager more responsive for real-world SOC operations.


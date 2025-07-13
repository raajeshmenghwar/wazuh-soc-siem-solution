## Manual Agent Registration Using `manage_agents`

To manually register an agent on the Wazuh Manager:

```bash
sudo /var/ossec/bin/manage_agents
````

Follow the on-screen menu:

* Press `A` to **add** a new agent (name, IP, and group).
* After adding, press `E` to **extract the agent key**.
* Save the key for use on the agent side.

---

## Agent Key Lifecycle

1. **Add Agent on Manager:**

   ```bash
   sudo /var/ossec/bin/manage_agents
   ```

2. **Extract the Key:**
   Select the agent → Extract key → Copy the key output.

3. **Import Key on Agent:**

   On the agent machine:

   ```bash
   sudo /var/ossec/bin/agent-auth -m <MANAGER_IP> -k '<AGENT_KEY>'
   ```

   Replace `<MANAGER_IP>` with your manager’s IP and `<AGENT_KEY>` with the extracted key string.

4. **Verify Connection:**

   On the manager:

   ```bash
   sudo tail -f /var/ossec/logs/ossec.log
   ```

---

## Automating Registration Using `authd`

### 1. Start `authd` Service on Manager:

```bash
sudo /var/ossec/bin/authd -p 1515 -A -v
```

| Flag | Description                            |
| ---- | -------------------------------------- |
| `-p` | Port to run `authd` on (default: 1515) |
| `-A` | Auto-add unknown agents                |
| `-v` | Verbose output (helpful for debugging) |

> **Note:** Ensure firewall allows port 1515.

---

### 2. Register Agent Automatically:

On the agent machine:

```bash
sudo /var/ossec/bin/agent-auth -m <MANAGER_IP>
```

The agent will request and receive its key automatically.

---

## Useful Agent Management Commands

| Command                        | Purpose                              |
| ------------------------------ | ------------------------------------ |
| `manage_agents`                | Add, remove, extract keys for agents |
| `agent-auth`                   | Agent-side registration utility      |
| `systemctl status wazuh-agent` | Check agent service status           |
| `journalctl -u wazuh-agent`    | View agent logs                      |

---

## Communication Ports

| Port   | Protocol | Description                     |
| ------ | -------- | ------------------------------- |
| `1515` | TCP      | Agent registration via `authd`  |
| `1514` | TCP/UDP  | Log forwarding to Wazuh Manager |

---

## Managing Existing Agents

To manage agents via CLI:

```bash
sudo /var/ossec/bin/manage_agents
```

* `A`: Add agent
* `E`: Extract agent key
* `R`: Remove agent
* `L`: List existing agents

> ⚠️ If an agent already exists, remove it first before re-adding.

---

## Security Recommendations

* Run `authd` only temporarily or restrict it to trusted IPs.
* Use firewall rules to secure access:

  ```bash
  sudo ufw allow from <TRUSTED_IP> to any port 1515
  ```

---

## Troubleshooting Tips

| Issue                  | Solution                                  |
| ---------------------- | ----------------------------------------- |
| Agent not connecting   | Ensure port 1515 is open and reachable    |
| Duplicate agent name   | Remove the old one and re-add             |
| Key expired or invalid | Re-extract and re-import the agent key    |
| No logs in dashboard   | Check if agent is active and sending logs |

---

## Alternative Method: Using Wazuh Dashboard

If using Wazuh 4.x+ with the built-in dashboard, agents can be registered via the UI:

1. Log in to the Wazuh Dashboard.
2. Go to **Agents → Add agent**.
3. Select platform (Linux, Windows, etc.).
4. Copy the generated command and run it on the agent system.
5. The agent will register automatically using the built-in key.

> This is the **easiest way** for beginners and is recommended for test environments or small setups.

---

For more details, see the official Wazuh documentation:
[Wazuh Agent Authentication Guide](https://documentation.wazuh.com/current/user-manual/agents/agent-auth.html)


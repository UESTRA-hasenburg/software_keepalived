# Ansible Rolle: software_keepalived

Diese Rolle installiert und konfiguriert Keepalived auf Debian 13 Systemen. Sie beinhaltet ein Notify-Skript zur Statusüberwachung und einen Checkmk Local Check.

## Funktionen

- Installation von `keepalived`.
- Konfiguration von VRRP-Instanzen über Templates.
- Automatische nftables-Regeln für VRRP-Traffic.
- **Keepalived Notify-Skript**: Speichert den aktuellen Status der VRRP-Instanzen in `/var/run/keepalived.*.state`.
- **Checkmk Integration**: Ein Python-Skript liest diese Statusdateien aus und stellt sie für Checkmk bereit.

## Nutzung des Notify-Skripts

Das Skript `keepalived_notify` wird automatisch in der `keepalived.conf` für jede VRRP-Instanz registriert:

```
vrrp_instance VI_1 {
    notify /etc/keepalived/keepalived_notify
    ...
}
```

Es sorgt dafür, dass bei jedem Statuswechsel (z.B. von BACKUP zu MASTER) eine Datei unter `/var/run/` aktualisiert wird, die vom Monitoring-Skript (`/usr/lib/check_mk_agent/local/keepalived`) ausgewertet werden kann.

## Konfigurationsbeispiele

Die Konfiguration erfolgt über die Variable `software_keepalived_vrrp_instances`.

### Beispiel 1: Einzelne VIP (String)

```yaml
software_keepalived_vrrp_instances:
  - name: VI_1
    id: 51
    priority: 100
    state: MASTER
    virtual_ip: "192.168.1.100/24"
```

### Beispiel 2: Mehrere VIPs (Liste)

```yaml
software_keepalived_vrrp_instances:
  - name: VI_CLUSTER
    id: 60
    priority: 150
    state: BACKUP
    virtual_ip:
      - "10.0.0.1/24"
      - "10.0.0.2/24"
```

## Überprüfung

Nach der Zuweisung der Rolle kann der Status der VRRP-Instanzen auf dem Zielsystem wie folgt geprüft werden:

- Keepalived Status: `systemctl status keepalived`
- Statusdateien: `ls /var/run/keepalived.*.state`
- Checkmk Output: `/usr/lib/check_mk_agent/local/keepalived`

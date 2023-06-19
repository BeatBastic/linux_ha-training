# Clusterlabs

Multi Node FailOver

- Erkennen und Wiederherstellen von Ausfällen
- Starten und Stoppen von Anwendungen in der richtigen Reihenfolge (auch Multi-Node)

Komponenten

- libQB - core service
- corosync - Messaging und Quorum
- Resource agents - Scripte für Services
- Fencing agents - Scripts für Network Power Switche und SAN devices zur Isolation von Cluster Servern
- Pacemaker - Überwachung uns Steierung von Anwendungen oder Diensten

Cluster Quorum

Cluster Resources

Installation

    apt update; apt install -y pacemaker corosync

Konfiguration

TCP ports 2224, 3121, and 21064, and UDP port 5405

<https://clusterlabs.org/pacemaker/doc/2.1/Clusters_from_Scratch/html/>

pcs service

    systemctl start pcs
    systemctl enable pcs

hacluster user

    passwd hacluster

Cluster Nodes authentifizieren

    pcs host auth <fqdn1> <fqdn2>
    pcs cluster setup <name> <fqdn1> <fqdn2>

pcs Kommandos

    cluster     Configure cluster options and nodes.
    resource    Manage cluster resources.
    stonith     Manage fence devices.
    constraint  Manage resource constraints.
    property    Manage pacemaker properties.
    acl         Manage pacemaker access control lists.
    qdevice     Manage quorum device provider on the local host.
    quorum      Manage cluster quorum settings.
    booth       Manage booth (cluster ticket manager).
    status      View cluster status.
    config      View and manage cluster configuration.
    pcsd        Manage pcs daemon.
    host        Manage hosts known to pcs/pcsd.
    node        Manage cluster nodes.
    alert       Manage pacemaker alerts.
    client      Manage pcsd client configuration.
    dr          Manage disaster recovery configuration.
    tag         Manage pacemaker tags.

Auslesen pacemaker Features

    pacemakerd --features

Cluster Starten

    pcs cluster start --all # oder
    systemctl start corosync
    systemctl start pacemaker

Corosync verifizieren

1. Kommunikation

    corosync-cfgtool -s

2. Mitglieder und Quorum

    corosync-cmapctl | grep members

Pacemaker verifizieren

    pcs status

Aktuelle Config auslesen

    pcs cluster cib

Config validieren

    pcs cluster verify --full

Fencing

Abschalten

    pcs property set stonith-enabled=false

Aktivieren

Power Fencing

- Power Switch
- IPMI
- Hardware Watchdog

Fabric Fencing

- Shared Storage
- Intelligente Netzwerk Switche

Fencing einrichten

Suchen nach Paketen mit dem Namen `*fence*`

    pcs stonith list
    pcs stonith describe <AGENT_NAME>
    pcs cluster cib stonith_cfg
    pcs -f stonith_cfg stonith create <STONITH_ID> <STONITH_DEVICE_TYPE> [STONITH_DEVICE_OPTIONS]
    pcs -f stonith_cfg property set stonith-enabled=true

Aktiv-Passiv Cluster

    pcs resource create ClusterIP ocf:heartbeat:IPaddr2 \
    ip=192.168.122.120 cidr_netmask=24 op monitor interval=30s

    pcs resource standards
    pcs resource providers
    pcs resource agents ocf:heartbeat

Resources schwenken

    pcs cluster stop <fqdn1>

Verhindern von Resource Relocation nach Wiederverfügbarkeit

    pcs resource defaults
    pcs resource defaults update resource-stickiness=100

Installation Apache

    apt install -y apache2

Einrichtung server-status (wird vom apache Resource Agent benötigt)

    # /etc/apache2/conf.d/status.conf
    <Location /server-status>
      SetHandler server-status
      Require local
    </Location>

Cluster Resource hinzufügen

    pcs resource create WebSite ocf:heartbeat:apache  \
      configfile=/etc/apache2/conf/apache2.conf \
      statusurl="http://localhost/server-status" \
      op monitor interval=1min

Timeout setzen

    pcs resource op defaults
    pcs resource op defaults update timeout=240s

Cluster Status prüfen

    pcs status

Webserver prüfen

    wget -O - http://localhost/server-status

Resourcen Abhängigkeiten

    pcs constraint colocation add WebSite with ClusterIP INFINITY
    pcs constraint
    pcs status

Resourcen Reihenfolgen

    pcs constraint order ClusterIP then WebSite
    pcs constraint

Cluster Node Präferenz

    pcs constraint location WebSite prefers <fqdn>=50
    pcs constraint
    pcs status

Placement Scores

    crm_simulate -sL

Resourcen Switchen

    pcs resource move WebSite <fqdn>
    pcs constraint
    pcs status

Weiter geht es mit [DRBD](../08_DRBD)
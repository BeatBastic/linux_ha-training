# OCFS2

## Installation

    apt install ocfs2-utils

## Initialsieren

    o2cb add-cluster ociocfs2

## Config

/etc/ocfs2/cluster.conf

## Nodes definieren

    o2cb add-node app1 initiator1 --ip 172.16.120.13
    o2cb add-node app2 initiator2 --ip 172.16.120.14

## Konfigurieren

    /sbin/o2cb.init configure

## Kernel Panic und OOPS

    sysctl kernel.panic=30
    sysctl kernel.panic_on_oops=1

## File System

    mkfs.ocfs2 -L "ocfs2" /dev/sdb

Weiter geht es mit [GlusterFS](../10_GlusterFS)
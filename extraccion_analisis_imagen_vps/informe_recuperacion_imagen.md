# Informe de extraccion

## Información General

- Fecha y hora de la extracción: 

03/04/2025 - 16:47

- Responsable de la extracción: 

Ismael

- Motivo de la extracción: 

Revisión del estado del vps

## Descripción del VPS

- IP pública: 

172.233.110.9
- IP privada
- Sistema Operativo y version: 

Kali GNU/Linux Rolling - 2024.4

- Recursos asignados: 

1 CPU Core

1 GB RAM

25 GB Storsage

- Aplicaciones o servicios en ejecucion: 
```
/sbin/init
[kthreadd]
[pool_workqueue_release]
[kworker/R-rcu_gp]
[kworker/R-sync_wq]
[kworker/R-slub_flushwq]
[kworker/R-netns]
[kworker/0:0H-kblockd]
[kworker/0:1-events]
[kworker/R-mm_percpu_wq]
[rcu_tasks_kthread]
[rcu_tasks_rude_kthread]
[rcu_tasks_trace_kthread]
[ksoftirqd/0]
[rcu_preempt]
[rcu_exp_par_gp_kthread_worker/0]
[rcu_exp_gp_kthread_worker]
[migration/0]
[idle_inject/0]
[cpuhp/0]
[kdevtmpfs]
[kworker/R-inet_frag_wq]
[kauditd]
[khungtaskd]
[oom_reaper]
[kworker/R-writeback]
[kcompactd0]
[ksmd]
[khugepaged]
[kworker/R-kintegrityd]
[kworker/R-kblockd]
[kworker/R-blkcg_punt_bio]
[irq/9-acpi]
[kworker/R-tpm_dev_wq]
[kworker/R-edac-poller]
[kworker/R-devfreq_wq]
[kworker/0:1H]
[kswapd0]
[kworker/R-kthrotld]
[kworker/R-acpi_thermal_pm]
[kworker/R-mld]
[kworker/R-ipv6_addrconf]
[kworker/R-kstrp]
[kworker/u5:0]
[scsi_eh_0]
[kworker/R-scsi_tmf_0]
[scsi_eh_1]
[kworker/R-scsi_tmf_1]
[kworker/R-ata_sff]
[scsi_eh_2]
[kworker/R-scsi_tmf_2]
[scsi_eh_3]
[kworker/R-scsi_tmf_3]
[scsi_eh_4]
[kworker/R-scsi_tmf_4]
[scsi_eh_5]
[kworker/R-scsi_tmf_5]
[scsi_eh_6]
[kworker/R-scsi_tmf_6]
[scsi_eh_7]
[kworker/R-scsi_tmf_7]
[jbd2/sda-8]
[kworker/R-ext4-rsv-conversion]
/usr/lib/systemd/systemd-journald
/usr/lib/systemd/systemd-udevd
[psimon]
[kworker/R-cryptd]
[watchdogd]
[kworker/R-ttm]
/usr/sbin/cron
/usr/bin/dbus-daemon
/usr/lib/systemd/systemd-logind
sshd:
/sbin/agetty
/sbin/agetty
/usr/sbin/chronyd
/usr/sbin/chronyd
sshd-session:
/usr/lib/systemd/systemd
(sd-pam)
[psimon]
sshd-session:
-zsh
[kworker/u4:1-events_unbound]
[kworker/u4:0-writeback]
/usr/bin/dbus-daemon
[kworker/0:0-cgroup_destroy]
[kworker/u4:2-events_unbound]
[psimon]
ps
awk
```

## Metodología

- Herramientas utilizadas

dd

gzip

- Comandos y parametros usados
```
ssh root@172.233.110.9 "sudo dd if=/dev/sda bs=4M | gzip -c" > vps_backup_lacabra.img.gz

sha256sum vps_backup_lacabra.img.gz 



```
## Integridad de la imagen

- Hash de la extracción
```
9337fd1cbe519be5ae9534d3bd0e3b0ea593019fa9ed3b3f45ea92dbe74f2ff4
```
## Firma de la imagen

- Algoritmo de hash usado

sha256

- Metodo de firma digital

RSA

- Clave publica del firmante



- Comandos usados para la firma
```
openssl genpkey -algorithm RSA -out privada.pem -pkeyopt rsa_keygen_bits:4096

openssl rsa -pubout -in privada.pem -out publica.pem

openssl dgst -sha256 -sign privada.pem -out firma.bin vps_backup_lacabra.img.gz

openssl dgst -sha256 -verify publica.pem -signature firma.bin vps_backup_lacabra.img.gz 
```
- Ubicacion de los archivos de firma y hash

https://limewire.com/d/VOYiX#Lvib3DZLwx
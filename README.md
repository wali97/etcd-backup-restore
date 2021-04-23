# etcd-backup-restore
etcd is a consistent and highly-available key value store used as  Kubernetes' backing store for all cluster data.
If your Kubernetes cluster uses etcd as its backing store, make sure you have a back up plan for those data.

_Before you begin:
You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster_

_Backing up an etcd cluster:
All Kubernetes objects are stored on etcd. Periodically backing up the etcd cluster data is important to recover Kubernetes clusters under 
disaster scenarios, such as losing all control plane nodes. The snapshot file contains all the Kubernetes states and critical information_
 
_Lets talk about etcd built-in snapshot
 so,before you backup u must know what all to backup...so backing up etcd means we have to backup all the certificates that are associated with clustercomponents
 and we've to backup & restore etcd server cluster.._
 
 $ ls /etc/kubernetes/pki/  		                                is default location where all the certificates r present
 $ ls /etc/kubernetes/pki/etcd/ 		                            is where all the etcd component certificates r present
 
_Now
 let's make a directory where we'll copy all the certificates.._
$ mkdir etcd-backup/      			                                this is where we'll copy all the certificates
$ cp -r /etc/kubernetes/pki/ etcd-backup/                       command to to copy all the certificates
$ ls etcd-backup/				                                        u can view the certificates if u wish to
 _so,the first part of bacup is done i.e backing up all the tls certificates & keys.
 
 
_now,we've to create a snapshot of etcd server...
 for that we need to have etcdctl tool installed..etcdctl is a command line tool for interacting with the etcd server._
 so let's install it:
 $ export RELEASE="3.3.13"
 $ wget https://github.com/etcd-io/etcd/releases/download/v${RELEASE}/etcd-v${RELEASE}-linux-amd64.tar.gz
 $ tar -xvf etcd-v3.3.13-linux-amd64.tar.gz
 $ cd etcd-v3.3.13-linux-amd64
 $ mv etcd etcdctl /usr/local/bin
 
 
_lets take snapshot:_
$ ETCDCTL_API=3 \						                                      etcdctl api version which we're using
$ etcdctl snapshot save etcd-backup/snapshot.db \        	        etcd-backup/snapshot.db is the file location where backup will be stored,u can give any name u want
$ --endpoints=https://127.0.0.1:2379 \				                    endpoint is nothing but the url on which etcd server is running\
$ --cacert=/etc/kubernetes/pki/etcd/ca.crt \			                ca certificates location.
$ --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \        tls certificate
$ --key=/etc/kubernetes/pki/etcd/healthcheck-client.key           key for tls cert

$ ls etcd-backup/   						                                  u can now see snapshot.db file there..which means u have successfully taken backup of etcd


_now,suppose u break this cluster somehow... 
how do u get it back?_

_lets restore etcd and get our cluster back_
so, first we need to copy all our certificates to /etc/kubernetes/
$ cp -r etcd-backup/pki /etc/kubernetes/

_now we've to restore etcd server_
$ ETCDCTL_API=3 \
$ etcdctl snapshot restore etcd-backup/snapshot.db     
the snapshot will be restored.

_but the snapshot is restore in /var/lib/etcd/ directory.._
so we need to move default.etcd/member to /var/lib/etcd/
$ls default.etcd/    						                                we need to copy the member directory to another part
$mv default.etcd/member /var/lib/etcd/ 

_ur last command to get ur cluster back as it was before. i.e. initializing ur cluster using kubeadm command_
$ kubeadm init --ignore-preflight-errors=DirAvailable--var-lib-etcd    which means that when our cluster is getting initialized it should ignore data for etcd server 									                                                     and take data from /var/lib/etcd/    fetch info from here.

$ kubectl get nodes					                                  	to view your cluster.

_ALL YOUR NODES & PODS WILL BE RESTORED AUTOMATICALLY & THE SERVICES RUNNING ON YOUR CLUSTER WILL CONTINUE RUNNING FROM WHERE U LEFT THEM.
THANK YOU_.

# Practice Test - Backup and Restore Methods 2
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-backup-and-restore-methods-2-2/)

Solutions to practice test - Backup and Restore Methods 2

In this test, we practice both with _stacked_ and _external_ etcd clusters.

1. Information only
1.  <details>
    <summary>Explore the student-node and the clusters it has access to.</summary>

    ```bash
    kubectl config get-contexts
    ```

    </details>
1.  <details>
    <summary>How many clusters are defined in the kubeconfig on the student-node?</summary>

    ```bash
    # get contexts and clusters in each context
    kubectl config get-contexts

    # get names of clusters
    kubectl config get-clusters

    # view details of each cluster
    kubectl config view
    ```

    > 2

    </details>
1.  <details>
    <summary>How many nodes (both controlplane and worker) are part of cluster1?</summary>

    ```bash
    kubectl config use-context cluster1
    kubectl get nodes
    ```

    > 2

    </details>
1.  <details>
    <summary>What is the name of the controlplane node in cluster2?</summary>

    ```bash
    kubectl config use-context cluster2
    kubectl get nodes
    ```

    > cluster2-controlplane

    </details>
1. Information only
1.  <details>
    <summary>How is ETCD configured for cluster1?</summary>

    ```bash
    kubectl config use-context cluster1
    kubectl get pods -n kube-system
    ```

    > From the output we can see a pod for etcd, therefore the answer is `Stacked ETCD`.
    > In a Stacked ETCD Topology the etcd storage cluster and the Kubernetes control plane share the same physical nodes. It contrasts with an External ETCD Topology, where etcd nodes are separate and dedicated only to running etcd, independent of the control plane nodes.

    </details>
1.  <details>
    <summary>How is ETCD configured for cluster2?</summary>

    ```bash
    kubectl config use-context cluster2
    kubectl get pods -n kube-system
    ```

    > From the output we can see no pod for etcd. Since no etcd is not an option for a _functioning_ cluster the answer must therefore be `External ETCD`
    > If you check out the pods running in the kube-system namespace in cluster2, you will notice that there are NO etcd pods running in this cluster!

student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  kubectl get pods -n kube-system  | grep etcd

student-node ~ ✖
Also, there is NO static pod configuration for etcd under the static pod path:

student-node ~ ➜  ssh cluster2-controlplane
Welcome to Ubuntu 23.10 (GNU/Linux 5.4.0-1106-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Thu Jul 25 19:31:58 2024 from 192.160.244.4

cluster2-controlplane ~ ➜  ls /etc/kubernetes/manifests/ | grep -i etcd

cluster2-controlplane ~ ✖ 
However, if you inspect the process on the controlplane for cluster2, you will see that that the process for the kube-apiserver is referencing an external etcd datastore:

cluster2-controlplane ~ ✖  ps -ef | grep etcd
root        2906    2515  0 19:26 ?        00:01:17 kube-apiserver --advertise-address=192.160.244.12 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.pem --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem --etcd-servers=https://192.160.244.3:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root        9228    8883  0 20:01 pts/0    00:00:00 grep etcd
You can see the same information by inspecting the kube-apiserver pod (which runs as a static pod in the kube-system namespace):

cluster2-controlplane ~ ➜  kubectl -n kube-system describe pod kube-apiserver-cluster2-controlplane 
Name:                 kube-apiserver-cluster2-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 cluster2-controlplane/192.160.244.12
Start Time:           Thu, 25 Jul 2024 19:26:33 +0000
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.160.244.12:6443
                      kubernetes.io/config.hash: 469506630571d428e4ddc39d3735a142
                      kubernetes.io/config.mirror: 469506630571d428e4ddc39d3735a142
                      kubernetes.io/config.seen: 2024-07-25T19:26:32.901498317Z
                      kubernetes.io/config.source: file
Status:               Running
SeccompProfile:       RuntimeDefault
IP:                   192.160.244.12
IPs:
  IP:           192.160.244.12
Controlled By:  Node/cluster2-controlplane
Containers:
  kube-apiserver:
    Container ID:  containerd://00e16e0d46214d1bdd969738966c660eb41a0f3a03ef0d2e0d8c009642381f82
    Image:         registry.k8s.io/kube-apiserver:v1.29.0
    Image ID:      registry.k8s.io/kube-apiserver@sha256:921d9d4cda40bd481283375d39d12b24f51281682ae41f6da47f69cb072643bc
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=192.160.244.12
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.pem
      --etcd-certfile=/etc/kubernetes/pki/etcd/etcd.pem
      --etcd-keyfile=/etc/kubernetes/pki/etcd/etcd-key.pem
      --etcd-servers=https://192.160.244.3:2379
--------- End of Snippet---------
    






    </details>
1.  <details>
    <summary>What is the IP address of the External ETCD datastore used in cluster2?</summary>

    For this, we need to exampine the API server configuration

    ```bash
    kubectl config use-context cluster2
    kubectl get pods -n kube-system kube-apiserver-cluster2-controlplane -o yaml | grep etcd
    ```

    > From the output, locate `--etcd-servers`. The IP address in this line is the answer.

    </details>
1.  <details>
    <summary>What is the default data directory used the for ETCD datastore used in cluster1?</summary>

    For this, we need to examine the etcd manifest on the control plane node, and we need to find out the hostpath of its `etcd-data` volume.

    ```bash
    kubectl config use-context cluster1
    kubectl get pods -n kube-system etcd-cluster1-controlplane -o yaml
    ```

    In the output, find the `volumes` section. The host path of the volume named `etcd-data` is the answer.

    > /var/lib/etcd

    </details>
1. Information only
1.  <details>
    <summary>What is the default data directory used the for ETCD datastore used in cluster2?</summary>

    For this, we need to examine the system unit file for the etcd service. Remember that for external etcd, it is running as an operating system service.

    ```bash
    ssh etcd-server
    ```

    ```bash
    # Verify the name of the service
    systemctl list-unit-files | grep etcd

    # Using the output from above command
    systemctl cat etcd.service
    ```

    Note the comment line in the output. This tells you where the service unit file is. We are going to need to edit this file in a later question!

    From the output, locate `--data-dir`

    > /var/lib/etcd-data

    Return to the student node:

    ```bash
    exit
    ```

    </details>
1.  <details>
    <summary>How many other nodes are part of the ETCD cluster that etcd-server is a part of?</summary>

    This question is somewhat contentious. It ought not to contain the word `other`. The required answer is

    > 1

    </details>
1.  <details>
    <summary>Take a backup of etcd on cluster1 and save it on the student-node at the path /opt/cluster1.db</summary>

    For this, we need to do the backup on the control node, then pull it back to the student node.

    ```bash
    ssh cluster1-controlplane
    ```

    ```bash
    ETCDCTL_API=3 etcdctl snapshot save \
      --cacert /etc/kubernetes/pki/etcd/ca.crt \
      --cert /etc/kubernetes/pki/etcd/server.crt \
      --key /etc/kubernetes/pki/etcd/server.key \
      cluster1.db

    # Return to student node
    exit
    ```

    ```bash
    scp cluster1-controlplane:~/cluster1.db /opt/
    ```

    </details>
1.  <details>
    <summary>An ETCD backup for cluster2 is stored at /opt/cluster2.db. Use this snapshot file to carry out a restore on cluster2 to a new path <b>/var/lib/etcd-data-new</b>.</summary>

    As you recall, `cluster2` is using _external_ etcd. This means
    * `etcd` does not have to be on the control plane node of the cluster. In this case, it is not.
    * `etcd` runs as an operating system service not a pod, therefore there is no manifest file to edit. Changes are instead made to a service unit file.</br></br>

    There are several parts to this question. Let's go through them one at a time.

    1.  <details>
        <summary>Move the backup to the etcd-server node</summary>

        ```bash
        scp /opt/cluster2.db etcd-server:~/
        ```
        </details>
    1.  <details>
        <summary>Log into etcd-server node</summary>

        ```bash
        ssh etcd-server
        ```

        </details>
    1.  <details>
        <summary>Check the ownership of the current etcd-data directory</summary>

        We will need to ensure correct ownership of our restored data. We determined the location of the data directory in Q12

        ```bash
        ls -ld /var/lib/etcd-data/
        ```

        > Note that owner and group are both `etcd`
        </details>
    1.  <details>
        <summary>Do the restore</summary>

        ```bash
        ETCDCTL_API=3 etcdctl snapshot restore \
            --data-dir /var/lib/etcd-data-new \
            cluster2.db
        ```

        </details>
    1.  <details>
        <summary>Set ownership on the restored directory</summary>

        ```bash
        chown -R etcd:etcd /var/lib/etcd-data-new
        ```

        </detials>
    1.  <details>
        <summary>Reconfigure and restart etcd</summary>

        We will need the location of the service unit file which we also determined in Q12

        ```bash
        vi /etc/systemd/system/etcd.service
        ```

        Edit the `--data-dir` argument to the newly restored directory, and save.

        Finally, reload and restart the `etcd` service. Whenever you have edited a service unit file, a `daemon-reload` is required to reload the in-memory configuration of the `systemd` service.

        ```bash
        systemctl daemon-reload
        systemctl restart etcd.service
        ```

        Return to the student node:

        ```bash
        exit
        ```

        </details>
    1.  <details>
        <summary>Verify the restore</summary>

        ```bash
        kubectl config use-context cluster2
        kubectl get all -n critical
        ```

        </details>
    </details>

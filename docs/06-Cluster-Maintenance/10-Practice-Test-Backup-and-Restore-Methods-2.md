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

    > From the output we can see no pod for etcd. Since no etcd is not an option for a _functioning_ cluster the answer must therefore be `External ETCD`.
    > The missing etcd doesn't directly imply it's an external etcd. It could be because of some problem with pod (static). To confirm we log into
    > cluster2-controlplane using `ssh cluster2-controlplane` and then check in `/etc/kubernetes/manifests` directory whether there is a file for etcd.
    > We don't find any files, meaning, etcd static pods isn't configured. Most likely now we believe it must be external. But to ensure that is the
    > correct assumption we use `kubectl describe pod kube-apiserver-cluster2-controlplane -n kube-system` and find under kube-apiserver
    > `--etcd-servers=https://192.160.244.3:2379`. The external url for the server confirms it's an external etcd.




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

     To verify the data directory for an external etcd service, you can either inspect the running process or check the system unit file. The quickest method is using `ps -ef | grep etcd` after SSH-ing into the etcd server. This command shows the currently running 
processes, including the `--data-dir` parameter used by etcd, making it ideal for a quick verification of the active state and runtime options.

    Alternatively, you can use `systemctl` to get a more comprehensive view. First, identify the service name with 
`systemctl list-unit-files | grep etcd`, then inspect the service configuration file using `systemctl cat etcd.service`. This approach not only shows the startup parameters but also provides the file paths and full configuration, which is useful if you plan to 
edit or troubleshoot further.

      The `kubectl describe` command did not work in this scenario because `kubectl` is designed to manage resources within the Kubernetes cluster, such as pods, nodes, and services. In an **External ETCD topology**, etcd is not deployed as a pod managed by Kubernetes; it is a standalone service running outside the cluster, often managed by `systemd` on a separate node. Therefore, accessing details about its configuration or state requires directly SSH-ing into the etcd server and using operating system-level commands (`ps`, `systemctl`), as Kubernetes commands like `kubectl` do not have access to external resources outside of the cluster’s control.

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

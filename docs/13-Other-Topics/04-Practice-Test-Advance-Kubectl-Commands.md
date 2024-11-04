# Practice Test for Advance Kubectl Commands

  - Take me to [Advance Practice Test for Kubectl Commands](https://kodekloud.com/topic/practice-test-advanced-kubectl-commands/)

  ### Solution

   1. Check Solution 

       <details>
       
        ```
        kubectl get nodes -o json > /opt/outputs/nodes.json
        ```   
       </details>

   2. Check Solution 

       <details>

        ```
        kubectl get node node01 -o json > /opt/outputs/node01.json
        ```   
       </details>

   3. Check Solution

       <details>

        ```
        kubectl get nodes -o=jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt
        ```
       </details>

   4. Check Solution

       <details>

        ```
        kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt
        ```
       </details>

   5. Check Solution

       <details>

        ```
        kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.users[*].name}" > /opt/outputs/users.txt
        ```
       </details>

   6. Check Solution

       <details>

        ```
        kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt
        ```

        The command `kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt` is used to sort PersistentVolumes (PVs) based on their storage capacity and output the sorted result to a file located at `/opt/outputs/storage-capacity-sorted.txt`.
        
        ### Explanation of Each Part
        
        - **`kubectl get pv`**: This retrieves the list of PersistentVolumes in the cluster.
        - **`--sort-by=.spec.capacity.storage`**: This option sorts the list of PVs by the specified field, in this case, `.spec.capacity.storage`, which refers to the storage capacity of each PV.
        - **`> /opt/outputs/storage-capacity-sorted.txt`**: Redirects the output of the command to a file for saving the sorted list.
        
        ### Why `--sort-by` Starts from `.spec` and Not from `.items`
        
        The **`--sort-by` option in `kubectl` specifies the path to the field within each individual resource object**, not the list as a whole. The reason it starts from `.spec` is that `kubectl` is accessing the properties of each **PersistentVolume** object individually within the list, not the `.items` array that holds all PVs in the JSON output.
        
        Here's how it works:
        
        - `.items` is a part of the JSON structure when viewing the entire output as a list of objects. However, `--sort-by` operates within each item (in this case, each PV), so it directly accesses `.spec.capacity.storage` within each object in the list.
        - If you used `.items`, `kubectl` would attempt to interpret `items` as a field in each individual PV object, which doesn’t exist, leading to an error.
        
        In summary, `--sort-by` refers to the path within each PV object itself, starting from `.spec.capacity.storage`.

       </details>

   7. Check Solution

       <details>

        ```
        kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage > /opt/outputs/pv-and-capacity-sorted.txt
        ```
       </details>

   8. Check Solution

       <details>

        ```
        kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" > /opt/outputs/aws-context-name
        ```
       </details>
       
       
       

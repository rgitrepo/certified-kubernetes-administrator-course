# Practice Test - Managing Application Logs
  - Take me to [Practice Test](https://kodekloud.com/topic/practice-test-managing-application-logs/)
  
Solutions to practice test - managing application logs
- We have deployed a POD hosting an application. Inspect it. Wait for it to start.

  <details>
  
  ```
  $ kubectl get pods
  ```
  </details>
  
- Inspect the logs of the POD
  
  <details>
  
  ```
  $ kubectl logs webapp-1
  ```
  </details>
  
- We have deployed a new POD - 'webapp-2' - hosting an application. Inspect it. Wait for it to start.

  <details>
  
  ```
  $ kubectl get pods
  ```
  </details>
  
- Inspect the logs of the webapp in the POD
  
  <details>
  
    Since the pod has two containers (simple-webapp & db) the below command might work but might also require container name to be specified for log viewing.

   Check the pod to find names of containers.
  
   ```
   $ kubectl describe pod webapp-2
   ```  



   ```
  $ kubectl logs webapp-2
  ```
   
  or

  ```
  $ kubectl logs webapp-2 simple-webapp
  ```
  
  </details>






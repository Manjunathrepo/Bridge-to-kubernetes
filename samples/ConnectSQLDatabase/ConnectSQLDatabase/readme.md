## <b>Connect SQL DB application</b>
**Motivation**
Sometimes customers may want to debug applications that talk to databases or other endpoints that are outside the cluster, but which are locked down for IP ranges specific to the cluster. In this case, it is normally impossible for them to debug the app locally without mocking this dependency. However, we offer a solution to them. When this endpoint is declared in the KubernetesLocalProcessConfig.yaml file, we proxy all calls to it via the cluster. This way, the customer is able to debug their application locally while communicating with the external endpoint.

**Hit an endpoint outside the cluster**
1. Create the SQL server by running `./DeploySqlServer` in a bash terminal. It should work without any changes.
2. Once the server has been created, deploy the sample app to your cluster: `kubectl apply -f Deployment.yaml`
3. Wait for the connect-sql-db pod to run: `kubectl get po --watch`
4. When the pod has come up, verify that it is able to talk to the database, e.g.: `kubectl logs connect-sql-db-7dbcdc4945-rdrh8`. If everything is correctly set up, the logs should look like:

```
DB_HOST = 'server-bridgetest123.database.windows.net'
Connection string: Data Source=server-bridgetest123.database.windows.net;Initial Catalog=database-bridgetest123;User ID=sampleLogin;Password=PLACEHOLDER

=================================
T-SQL to 2 - Create-Tables...
-1 = rows affected.

=================================
T-SQL to 3 - Inserts...
8 = rows affected.

=================================
T-SQL to 4 - Update-Join...
2 = rows affected.

=================================
T-SQL to 5 - Delete-Join...
2 = rows affected.

=================================
Now, SelectEmployees (6)...
d9bf50a1-700e-490f-a999-d0ec7857fa3d , Alison , 20 , acct , Accounting
916aea90-52d7-4c64-8118-548c46760802 , Barbara , 17 , hres , Human Resources
2f6f0745-38d7-42ef-a490-a92374c421f5 , Carol , 22 , acct , Accounting
a21c384a-6af3-420e-9cba-cbf2bec75560 , Elle , 15 , NULL , NULL
Going to sleep...
```
5. Open ConnectSQLDatabase in VSCode. If you try to run the sample app locally, it should crash with an "unauthorized" error message.
    - You may need to create a ".NET Core Launch (console)" launch profile in launch.json if it is your first time running the app. 
6. Configure with Bridge (ctrl+shift+P, `Bridge to Kubernetes: Configure`)
    - Service: connect-sql-db
    - Port: 5000
    - Launch profile: ".NET Core Launch (console)"
    - Isolated: no
7. Debug the app. You should be able to see the same output as you did in the pod (see step #4).
8. Clean up the server: `az group delete -n SqlServerRG`

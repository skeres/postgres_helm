# resources
https://artifacthub.io/packages/helm/bitnami/postgresql
https://github.com/bitnami/charts/tree/main/bitnami/postgresql/


# add repo
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo update

# tip for auto complete in bash
$ echo 'source <(kubectl completion bash)' >>~/.bashrc              # at the next connexion to terminal, it will be active
$ source <(kubectl completion bash)                                 # activate it right now, but not in other terminal

# search in repo
$ helm search repo postgresql
result : 
NAME                 	CHART VERSION	APP VERSION	DESCRIPTION                                       
bitnami/postgresql   	12.1.2       	15.1.0     	PostgreSQL (Postgres) is an open source object-...
bitnami/postgresql-ha	10.0.4       	15.1.0     	This PostgreSQL cluster solution includes the P...

# pull the chart : retrieve a package from a package repository, and download it locally.
$ helm pull bitnami/postgresql --version 12.1.2


helm pull [chart URL | repo/chartname] [...] [flags]
Options
      --ca-file string             verify certificates of HTTPS-enabled servers using this CA bundle
      --cert-file string           identify HTTPS client using this SSL certificate file
  -d, --destination string         location to write the chart. If this and untardir are specified, untardir is appended to this (default ".")
      --devel                      use development versions, too. Equivalent to version '>0.0.0-0'. If --version is set, this is ignored.
  -h, --help                       help for pull
      --insecure-skip-tls-verify   skip tls certificate checks for the chart download
      --key-file string            identify HTTPS client using this SSL key file
      --keyring string             location of public keys used for verification (default "~/.gnupg/pubring.gpg")
      --pass-credentials           pass credentials to all domains
      --password string            chart repository password where to locate the requested chart
      --prov                       fetch the provenance file, but don't perform verification
      --repo string                chart repository url where to locate the requested chart
      --untar                      if set to true, will untar the chart after downloading it
      --untardir string            if untar is specified, this flag specifies the name of the directory into which the chart is expanded (default ".")
      --username string            chart repository username where to locate the requested chart
      --verify                     verify the package before using it
      --version string             specify a version constraint for the chart version to use. This constraint can be a specific tag (e.g. 1.1.1) or it may reference a valid range (e.g. ^2.0.0). If this is not specified, the latest version is used

# untar compress file
$ tar zxvf postgresql-12.1.2.tgz -C .  ( "." is the current directory. You van set your own path )


# playing the game
helm template ./postgresql : show yml files that will be generated
helm install postgres-test  --debug --dry-run ./postgresql : to simulate a deployment

# For debugging, create a busybox POD and ssh into it
kubectl run -i --tty debug --image=busybox --restart=Never -- sh
kubectl exec -it debug -- sh
inside the pod : execute any command you want
kubectl delete -n default pod debug

alternate busybox tool : 
source : https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox sh

$ minikube start --cpus=4 --memory=6000 --kubernetes-version=v1.25.2

$ minikube ssh
docker@minikube:~$ sudo mkdir /mnt/data-postgres-k8s
socker@minikube:~$ sudo chmod 777 data-postgres-k8s
docker@minikube:~$ exit
$ kubectl apply -f postgresql/volumes/postgres-pv.yaml  : create PV FIRST
$ kubectl apply -f postgresql/volumes/postgres-pvc.yaml : create PVC

to remove volumes ( in kubernetes, not on your mount path ): DELETE PVC first !
$ kubectl delete persistentvolume postgresql-pvc
$ kubectl delete persistentvolume postgresql-pv

# adjust your parameters in value.yml. check for label ">>> update" to set your own values
Example : 
auth:
      postgresPassword: "postgres"
      username: "stephane"
      password: "postgres"

$ helm install postgres-test ./postgresql

----------------------------------------------------------------------------
Result : 

NAME: postgres-test
LAST DEPLOYED: Tue Nov 22 20:35:51 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 12.1.2
APP VERSION: 15.1.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    postgres-test-postgresql.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_ADMIN_PASSWORD=$(kubectl get secret --namespace default postgres-test-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To get the password for "stephane" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgres-test-postgresql -o jsonpath="{.data.password}" | base64 -d)

To connect to your database run the following command:

    kubectl run postgres-test-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:15.1.0-debian-11-r0 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgres-test-postgresql -U stephane -d postgres -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/postgres-test-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U stephane -d postgres -p 5432


----------------------------------------------------------------------------

Next steps : to connect to your database from outside the cluster execute the following commands:
in a separate terminal and run : 
$ kubectl port-forward --namespace default svc/postgres-test-postgresql 5432:5432
come back to your main terminal and run : 
$ psql --host 127.0.0.1 -U stephane -d postgres -p 5432  
result : 
Password for user stephane: 
psql (14.5 (Ubuntu 14.5-0ubuntu0.22.04.1), server 15.1)
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
Type "help" for help.

postgres=>
postgres=> \l                                # list all existing database
postgres=> create DATABASE mylittledb;       # create your own database
postgres-> \l
result : 
                                  List of databases
    Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
------------+----------+----------+-------------+-------------+-----------------------
 mylittledb | stephane | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres   | stephane | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/stephane         +
            |          |          |             |             | stephane=CTc/stephane
 template0  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
            |          |          |             |             | postgres=CTc/postgres
 template1  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
            |          |          |             |             | postgres=CTc/postgres
(4 rows)

# To delete databse : 
postgres=> drop DATABASE mylittledb;

postgres=> exit ( or \q to quit )

End of game : 
$ helm uninstall postgres-test
$ minikube stop

-------------------------------------------------------------------

Playing with AZURE AKS

sources
https://learn.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv

The following example shows the pre-create storage classes available within an AKS cluster
bash-5.1# kubectl get sc

Note : Persistent volume claims are specified in GiB but Azure managed disks are billed by SKU for a specific size. 
These SKUs range from 32GiB for S4 or P4 disks to 32TiB for S80 or P80 disks (in preview).

Create a persistent volume claim

see storage class available
result :  
NAME                    PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
azurefile               file.csi.azure.com   Delete          Immediate              true                   4d3h
azurefile-csi           file.csi.azure.com   Delete          Immediate              true                   4d3h
azurefile-csi-premium   file.csi.azure.com   Delete          Immediate              true                   4d3h
azurefile-premium       file.csi.azure.com   Delete          Immediate              true                   4d3h
default (default)       disk.csi.azure.com   Delete          WaitForFirstConsumer   true                   4d3h
managed                 disk.csi.azure.com   Delete          WaitForFirstConsumer   true                   4d3h
managed-csi             disk.csi.azure.com   Delete          WaitForFirstConsumer   true                   4d3h
managed-csi-premium     disk.csi.azure.com   Delete          WaitForFirstConsumer   true                   4d3h
managed-premium         disk.csi.azure.com   Delete          WaitForFirstConsumer   true                   4d3h

Create PVC
bash-5.1# kubectl apply -f azure-pvc.yaml 
result : 
persistentvolumeclaim/azure-managed-disk created

Display existing PVC
bash-5.1# kubectl get pvc
NAME                 STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
azure-managed-disk   **Pending**                                     managed-csi    5m23s

Deploy postgres
bash-5.1# helm install postgres-test ./postgresql_azure

bash-5.1# kubectl get pvc
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
azure-managed-disk   Bound    pvc-f42a6b74-c5aa-4cab-a6a9-556a30493b7d   1Gi        RWO            managed-csi    17m

check
bash-5.1# helm list
NAME         	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART            	APP VERSION
postgres-test	default  	1       	2022-12-09 21:44:08.688809447 +0000 UTC	deployed	postgresql-12.1.2	15.1.0     

bash-5.1# kubectl get service 
NAME                          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubernetes                    ClusterIP   10.0.0.1      <none>        443/TCP    4d4h
postgres-test-postgresql      ClusterIP   10.0.242.75   <none>        5432/TCP   5m4s
postgres-test-postgresql-hl   ClusterIP   None          <none>        5432/TCP   5m4s

bash-5.1# kubectl get pod -o wide
NAME                                          READY   STATUS    RESTARTS   AGE   IP            NODE                                NOMINATED NODE   READINESS GATES
postgres-test-postgresql-0                    1/1     Running   0          40m   **10.244.1.15**   aks-nodepool1-22173517-vmss000000   

connect into the pod and create database "mylittledb"
bash-5.1# kubectl exec -it postgres-test-postgresql-0 -- bash

$ psql --host 10.244.1.15 -U stephane -d postgres -p 5432 

postgres=> create DATABASE mylittledb;       # create your own database
postgres=> \l                                # list all existing database
result : 
                                  List of databases
    Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
------------+----------+----------+-------------+-------------+-----------------------
 mylittledb | stephane | UTF8     | en_US.UTF-8 | en_US.UTF-8 | ( truncated for safety)

 postgres=> \q                                 # quit psql

deploy springbootAPI2 and go into pod, then try : 

 curl http://springboot-k8s-svc:8484/api/listeEtudiants
 
[{"id":1,"nom":"k","prenom":"steph","dateNaissance":"2022-12-09T22:56:07.179+00:00"},{"id":2,"nom":"m","prenom":"maria","dateNaissance":"2022-12-09T22:56:07.557+00:00"},{"id":3,"nom":"p","prenom":"phil","dateNaissance":"2022-12-09T22:56:07.574+00:00"},{"id":4,"nom":"j","prenom":"jessica","dateNaissance":"2022-12-09T22:56:07.628+00:00"},{"id":5,"nom":"z","prenom":"jessica","dateNaissance":"2022-12-09T22:56:07.678+00:00"},{"id":6,"nom":"k","prenom":"steph","dateNaissance":"2022-12-09T23:24:53.705+00:00"},{"id":7,"nom":"m","prenom":"maria","dateNaissance":"2022-12-09T23:24:54.323+00:00"},{"id":8,"nom":"p","prenom":"phil","dateNaissance":"2022-12-09T23:24:54.347+00:00"},{"id":9,"nom":"j","prenom":"jessica","dateNaissance":"2022-12-09T23:24:54.368+00:00"},{"id":10,"nom":"z","prenom":"jessica","dateNaissance":"2022-12-09T23:24:54.381+00:00"}]

it works !







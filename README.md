***Using Helm and Operators in Kubernetes***

Today we are going to take a look at using the Kubernetes Package Manager called as Helm and also taste the magic of Operators.

Make sure Docker for Desktop is running.

Open up a terminal and first make sure you dont have any old clusters running, if you do , then please delete them using

    kind get clusters
    kind delete cluster --name tambootcamp

Create the KinD cluster using

    kind create cluster --name tambootcamp

Give it some time, make sure the nodes status shows Ready using

    kubectl get nodes

Install helm on a mac

    brew install helm

On Windows 

    choco install kubernetes-helm

Just like everything else, you can get help using 

    helm --help

Look at the version of helm using 

    helm version

Let's add a repository now using 

    helm repo add bitnami https://charts.bitnami.com/bitnami

You can add as many repositories you want and can list them using 

    helm repo ls

You can search the repositories for any applications you would like to deploy. Today I would like to use kubeapps in the kubeapps namespace

I will first create a namespace called as kubeapps using 

    kubectl create ns kubeapps

Next I will install the Helm chart for it using

    helm install kubeapps --namespace kubeapps bitnami/kubeapps

Give it a moment, you will see the chart being deployed. 

Let's port-forward as directed from the output

    kubectl port-forward --namespace kubeapps service/kubeapps 8080:80

To access kubeapps, you will need a token, ideally an K8S API token but just for our KinD cluster , we can create a service account and create a clusterrolebinding for it and then extract the token out of it. 

First stop the port-binding(Ctrl-C) since you need to do all that we talked above!


To create a service account:

    kubectl create serviceaccount kubeapps-operator

Now create a ClusterRoleBinding:

    kubectl create clusterrolebinding kubeapps-operator --clusterrole=cluster-admin --serviceaccount=default:kubeapps-operator

Now extract the token from the secret created:

    kubectl get secret $(kubectl get serviceaccount kubeapps-operator -o jsonpath='{range .secrets[*]}{.name}{"\n"}{end}' | grep kubeapps-operator-token) -o jsonpath='{.data.token}' -o go-template='{{.data.token | base64decode}}' && echo

Now do a port-forward like before using the command below

    kubectl port-forward --namespace kubeapps service/kubeapps 8080:80

Copy the token to authenticate on the browser to see the page. Have a look around. It's pretty cool stuff, lot's of things to try in there. 

By the way, you can deploy applications from kubeapps directly onto your kubernetes clusters. 

Even though most of the time you will not be creating new charts, you should know that helm is also a templating engine. Helm helps reinforce the Configuration principles of the 12 Factor App Methodology, where by we are separating configuration information from the application. 

Let's take a look at deploying a MariaDB helm chart, look at the --set option which I am passing along to affect the deployment of MariaDB. 

The parameters which you can change can be seen at https://github.com/bitnami/charts/tree/master/bitnami/mariadb 


    helm install mariadb bitnami/mariadb --set rootUser.password=PASSWORD


Here's how you would get the administrator's password(copy it into your clipboard right now(not the %), we will need it!)

    kubectl get secret --namespace default mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 --decode

Deploy a client for the DB you just deployed using 

    kubectl run mariadb-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mariadb:10.5.10-debian-10-r18 --namespace default --command -- bash

When asked for the password. Paste the password which you had copied. There you go, you just deployed a database and a client to connect it. 

Let's put some data into it and see if it actually works! Remember database queries end with a ";"

    USE my_database;
    CREATE TABLE accounts (id INT NOT NULL, username VARCHAR(255) NOT NULL);
    INSERT INTO accounts VALUES (1, 'darth'), (2, 'vader');
    SELECT * FROM accounts;

There you go you should see 

    2 rows in set (0.002 sec)

Type '*exit*' to get out of the database and then once more '*exit*' to exit out of the client pod. You will notice that the client pod gets deleted.

Now let's look at Operators. 
You can install operators using helm! 

For this, we will use prometheus as an example. 

    helm install myprom bitnami/kube-prometheus

Alternately, you could use kubeapps to do this through the UI! I like that approach too. 

Prometheus is a big application, installing things manually and in the right order could be daunting and hence operators come to rescue. 

After some time, you can port-forward the prometheus pod to have a look at localhost:9090

    kubectl port-forward --namespace default svc/myprom-kube-prometheus-prometheus 9090:9090

There you go, you deployed the prometheus application. Explore the page at localhost:9090

To remove this, it is as easy as

    helm uninstall myprom

That's it for today folks! See you next time :) 




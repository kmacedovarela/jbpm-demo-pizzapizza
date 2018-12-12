jBPM Sample - Pizza Pizza Demo
=============================

This is a demo business application that demonstrates decoupled low code development for business processes and rule using jBPM. Usage sample with images at the end of this file.

jBPM is open source software, released under the Apache Software License. It is written in 100% pure Java™, runs on any JVM and is available in the Maven Central repository too. 

In this sample, Google Assistant can be used to start a pizza order (bpmn2 process), which calculates the price and delivery time according to rules (drools decision tables) and waits for an attendant approval. Google Assistant informs the customer of the price and deliver time. After the approval from the pizza service, the processes defines if the delivery service should be activeted, or if an email should be sent to the customer.

The whole project was authored on business central and developed using dialogflow console.

The bpmn process uses a `Remote Business Rule Task`, this means the [process project](https://github.com/kmacedovarela/pizzapizza-demo/tree/master/jbpm-processes-pizzapizza) is deployed independently from the [rules project](https://github.com/kmacedovarela/pizzapizza-demo/tree/master/jbpm-rules-pizzapizza), what makes changes and promotion of rule deployments easier and independent from process lifecycle facilitating business changes. Both projects relies on the same [model](https://github.com/kmacedovarela/pizzapizza-application-model), Order.java. 

## Installation

To test this project its necessary to start jBPM, import projects, create its respective two kie containers and import the google assistant project to dialogflow. 

*For now, this file describes a local install without docker*

### jBPM 

1. Download jBPM 7.15 full installer of [jBPM 7.15](https://download.jboss.org/jbpm/release/7.15.0.Final/jbpm-installer-full-7.15.0.Final.zip)
1. Start jBPM with the following command
   1. `$PATH_TO/jbpm-server-7.14.0.Final/bin/standalone.sh -Dorg.kie.server.xstream.enabled.packages='org.kvarela**` 

## Importing processes and rules 

After your jBPM is up and running you can import the project via jbpm-console or via rest API. 

For the sake of this demo, here are the curl commands to import and built the three projects:

*Remember to replace `localhost:8080` if a different socket for the service has been used*.

1. Import the model, rules and process projects to default space `MySpace`: 

**Model project:**

~~~

curl --request GET  --header 'Authorization: Basic c2FsYWJveTpzYWxhYm95'  --url http://localhost:8080/jbpm-console/rest/spaces/MySpace/projects

curl --request POST --url http://localhost:8080/jbpm-console/rest/spaces/MySpace/git/clone --header 'Authorization: Basic c2FsYWJveTpzYWxhYm95' --header 'Content-Type: application/json' --header 'cache-control: no-cache' --data '{"name":"pizzapizza-application-model","description":"Pizza Pizza - Model project.","gitURL":"git@github.com:kmacedovarela/pizzapizza-application-model.git"}'

curl --request POST   --url http://localhost:8080/jbpm-console/rest/spaces/MySpace/projects/pizzapizza-application-model/maven/install  --header 'Authorization: Basic c2FsYWJveTpzYWxhYm95'

~~~

**Rules project:**

~~~

curl --request POST --url http://localhost:8080/jbpm-console/rest/spaces/MySpace/git/clone --header 'Authorization: Basic c2FsYWJveTpzYWxhYm95' --header 'Content-Type: application/json' --header 'cache-control: no-cache' --data '{"name":"jbpm-rules-pizzapizza","description":"Pizza Pizza JBPM Demo - Rules project.","gitURL":"git@github.com:kmacedovarela/jbpm-rules-pizzapizza.git"}'


curl --request POST   --url http://localhost:8080/jbpm-console/rest/spaces/MySpace/projects/jbpm-rules-pizzapizza/maven/install  --header 'Authorization: Basic c2FsYWJveTpzYWxhYm95'

~~~

**Processes project:**

~~~

curl --request POST   --url http://localhost:8080/jbpm-console/rest/spaces/MySpace/git/clone   --header 'Authorization: Basic c2FsYWJveTpzYWxhYm95' --header 'Content-Type: application/json' --header 'cache-control: no-cache' --data '{"name":"pizzapizza-processes-kjar","description":"Pizza Pizza JBPM Demo - Process project.","gitURL":"git@github.com:kmacedovarela/jbpm-processes-pizzapizza.git"}'

curl --request POST   --url http://localhost:8080/jbpm-console/rest/spaces/MySpace/projects/jbpm-processes-pizzapizza/maven/install  --header 'Authorization: Basic c2FsYWJveTpzYWxhYm95'

~~~

## Deploying kie containers

1. Access your jbpm-console console (Ex. http://localhost:8080/jbpm-console) and check that the three projects were imported as expected. 

1. Open the `pizzapizza-rules-kjar` project, and click on deploy. jBPM will build and deploy the project on Kie Server. A new kie container will be created to process the rules in this project;

1. Open the `pizzapizza-process-kjar` project, and click on deploy. This will create a new kie container to manage the process instances;

*These tasks can also be automated via REST. Still on #TODO list*

## Creating the Google Assistant project

The [dialogflow-nodejs-pizzapizza](https://github.com/kmacedovarela/dialogflow-nodejs-pizzapizza/tree/88a5c6a158412eb277ee3e85ea902a05a3eb8e67) project, contains a zip file that has the intents and fullfilment configurations. All it's required to do is import this zip file into a dialogflow project.

It contains also a primitive node.js script used by google assistant to reach the Kie Server via rest API. On this demo, the script runs on the cloud with firebase functions, and tries to reach the Kie Server http address. 

Since google assistant runs on the internet, you need to expose your kie containers urls to so the node.js client on firebase can be able to reach it. For local tests, you can use [ngrok](https://ngrok.com/) to create a tunnel and expose kie server. Example:

~~~

./ngrok http 8080

~~~ 

When you have your external access url, it's necessary to update the node.js client and redeploy it:

1. Setup the url on the [index.js](https://github.com/kmacedovarela/dialogflow-nodejs-pizzapizza/blob/88a5c6a158412eb277ee3e85ea902a05a3eb8e67/functions/index.js#L15) file. 
1. Access `dialogflow-nodejs-pizzapizza/functions` and deploy the project on firebase. Here are some tips on firebase usage:

- `firebase list` - List your available projects

- `firebase deploy --project pizza-order-XPT0` - Deploy the new version of your script


# Using it

Start a simulation on your dialogflow console and enjoy ordering pizza. 

![Image of Google Assistant](https://github.com/kmacedovarela/jbpm-demo-pizzapizza/blob/master/images/google-assistant.png)

Google assistant started a new process via the Node JS client available on firebase. The NodeJS client executed a rest call to one of the kie containers with the input from the user. Verify that a process was started and is waiting for a human approval.

![Image of Decision Table](https://github.com/kmacedovarela/jbpm-demo-pizzapizza/blob/master/images/order-process-instance.png)

The price and value informed to the customer, were calculated dinamicaly by remote decision tables deployed in a separate kie container (could be in a different docker container).

The human tasks forms can be developed using business central and used in client application if needed. Since this is not main focus of this demo, the form is really simple for now:

![Image of Decision Table](https://github.com/kmacedovarela/jbpm-demo-pizzapizza/blob/master/images/task-forms.png)

Change rules values on the decision tables, deploy the project and order another pizza.

![Image of Decision Table](https://github.com/kmacedovarela/jbpm-demo-pizzapizza/blob/master/images/price-rules.png)

When implementing business proccesses and rules using proper technologies and methodologies, business requirements can be quickly achieved with little effort from dev and ops teams. It always important to remember that all kpis are being stored to be presented on a friendly dashboard for business users. 

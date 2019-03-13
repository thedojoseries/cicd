# Challenge

In this challenge, you will to build an elaborated Continuous Integration / Continuous Deployment pipeline using [Jenkins](https://jenkins.io/), one of the most popular CI/CD tools. In summary, this pipeline will do the following actions:

* Checkout a GitHub repository containing a simple Node.js application
* Run a few tests
* Build a Docker image and push it to Dockerhub
* Deploy the application to Kubernetes using Helm (Kubernetes' package manager)

To simplify the process, you will follow a number of steps. Each step will help you build a tiny bit of the final solution.

# Jenkins on Kubernetes

We usually run Jenkins on virtual machines and provide it with a script which is going to run on the host. However, for this challenge, Jenkins will be running on a Kubernetes cluster, which means that all scripts will have to run in containers. This is a bit of a shift of paradigm, but there are good reasons for that approach. Read them below **if you have the time**, otherwise, proceed to the **Getting started** section. 

## Why run Jenkins on a container orchestration platform? (Optional section - read at your own free time)

There are a couple of reasons why running pipelines in containers is a better approach. Let's suppose that Jenkins is running on a single virtual machine, and is being managed by an operations team. The developers have just started developing a new Node.js application and ask the operations team to install NPM on the Jenkins host. The operations team install it. A month later, they start developing a microservice written in Python and therefore need Pip. Developers ask the operations team to install pip on Jenkins for their new python application. A month later, they start developing a Java application. You know what's going to happen, right? There's just too much unnecessary work for the operations team.

Things can get much worse if the operations team decide to run Jenkins in a Master-Worker fashion (one Master node and multiple worker nodes). If before it was challenging to manage one host (Jenkins Master), imagine multiple hosts.

How can this dev and ops relationship be improved? By introducing containers. In this new scenario, whenever developers start developing some new application or decide to improve existing applications, they should provide the CI/CD pipeline with a Dockerfile which builds a Docker image and run everything that needs to be run. This means that if there is a Node.js application with dependencies, all the tools necessary to test and build the application will be installed **in the container**. This means that developers do not need to come to the operations team anymore and ask them to install tools in the virtual machines. Developers only need to provide the ops team with a Dockerfile. The ops team does not need to worry about which tools and commands are needed to build the application.

# Getting started

The **base infrastructure** consists of a Kubernetes cluster running Jenkins. For the most part of this challenge, you will only interact with Jenkins. Whenever you need to interact with Kubernetes, commands will be provided.

To make sure that everyone has the same environment, you will spin up a Docker container which will contain all the tools necessary for you to solve this challenge.

Issue the following Docker command:

```
docker run --rm -it -e "URL1=<URL1>" -e "URL2=<URL2>" thedojoseries/env:cicd
```

The command above will spin up a container using the *thedojoseries/env* image and tag *cicd* (each Dojo has its own tag for the thedojoseries/env image). You will also notice that you need to specify two environment variables: **URL1** and **URL2**. You will be given further instructions at the beginning of the challenge. If you Docker command fails, please reach out to one of the organizers.

Once you are in the container, you will need to use **kubectl** to interact with Kubernetes and retrieve Jenkins' IP address. The command below will retrieve the IP address, but wait! Before you run the command, note that it contains multiple occurrences of $$the string `jenkins-team<X>`. Replace `<X>` with the number of your team (e.g. Team 1 should `jenkins-team1`, Team 2 should be `jenkins-team2` etc). Once `<X>` has been replaced, run the command:

```
kubectl get svc jenkins-team<X> -n jenkins-team<X> -o jsonpath='{.status.loadBalancer.ingress[0].ip}' && echo :8080
```

Now that you have Jenkins' IP address (Jenkins is running on port **8080**), copy and paste the IP in your Browser so you can access Jenkins.
When you see the interface, you will be asked for a username and password. The username is **admin**. To obtain the password, go back to the container and run the following command (replacing `<X>` with the number of your team again): 

```
printf $(kubectl get secret --namespace jenkins-team<X> jenkins-team<X> -o jsonpath="{.data.jenkins-admin-password}" | base64 -d);echo
```

Log in to Jenkins. Any issues, please let one of the organizers know.

# Step 1: Fork the thedojoseries/cicd-challenge-application repository

First, log in to your own GitHub account. Then, head over to [https://github.com/thedojoseries/date-display-app](https://github.com/thedojoseries/date-display-app) and click the **Fork** button at the top-right corner. This will create a copy of the date-display-app repository in your GitHub account. If you do not know what Fork means, [read this](https://help.github.com/articles/fork-a-repo/)

**This step is necessary because later on you will have to push code to the repository and you will not be able to push it to thedojoseries's repository.** 

# Step 2: Create a Dockerhub account if you do not have one

Dockerhub is like GitHub, but for storing Docker containers. If any member of your team already has an account, proceed to the next step. Otherwise, visit [https://hub.docker.com/](https://hub.docker.com/) and create a new account.

# Step 3: Create a Pipeline job

The first step will be to create a pipeline job. Head over to the Jenkins interface and click on **New Item** (left panel) and create the pipeline (give it any name you wish). Note that the item type needs to be **Pipeline** (don't select *Freestyle Project* or *Multibranch Pipeline*). You will configure the pipeline in the next step.

# Step 4: Configure the pipeline

Once the Pipeline job is created, you will have to configure it. If you have never configured a Jenkins job before, you will see that there are a lot of things to be configured. We will just be focusing on what's important for this challenge. 

A Jenkins pipeline is defined with code (using the [Groovy programming language](http://groovy-lang.org/)). You can write the code either directly on Jenkins' interface or in a file. As a best practice, you will want to write the code in a file and store it in a remote repository. You can choose any file name you'd like, but usually we use **Jenkinsfile** as the name. This Jenkinsfile can be kept together with the application code, in the same repository.

After you create the job, go to the **Pipeline** section. You will see a little text editor. This is so you can write the pipeline code there. Ignore the text editor and click on **Definition** and select **Pipeline script from SCM**. In *SCM*, select **Git**. Paste the URL of your repository in *Repository URL* (use the HTTPS version). Since your repository is public, you don't have to worry about the *Credentials*. In *Branch Specifier*, set it to `*/master`. In *Script Path*, set it to **Jenkinsfile**. Finally, click **Save**.

# Step 5: Build the pipeline the first time

The repository you forked on Step 1 already contains a Jenkinsfile. Therefore, you can build the pipeline and see how it works. Click on **Build Now**. Once the pipeline starts building, click on the #1 row (left-hand side of the page), then click on **Console Output**. You should see the following output:

```
Started by user admin
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] node
Still waiting to schedule task
Waiting for next available executor
Agent default-cn1t0 is provisioned from template Kubernetes Pod Template
Agent specification [Kubernetes Pod Template] (jenkins-team1-jenkins-slave ): 
* [jnlp] jenkins/jnlp-slave:3.10-1(resourceRequestCpu: 500m, resourceRequestMemory: 1024Mi, resourceLimitCpu: 500m, resourceLimitMemory: 1024Mi)

Running on default-xxxxx in /home/jenkins/workspace/my-pipeline
[Pipeline] {
[Pipeline] echo
Your Pipeline works!
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

Congrats, your pipeline works! Now, let's start writing some code, shall we?!

# Step 6: Familiarize yourself with a few Jenkins Pipeline concepts (Optional)

[What is a Pipeline?](https://jenkins.io/doc/book/pipeline/#pipeline)

[What is a Node?](https://jenkins.io/doc/book/pipeline/#node)

[What is a Stage?](https://jenkins.io/doc/book/pipeline/#stage)

[What is a Step?](https://jenkins.io/doc/book/pipeline/#step)

[Declarative Pipeline](https://jenkins.io/doc/book/pipeline/#declarative-pipeline-fundamentals)

[Scripted Pipeline](https://jenkins.io/doc/book/pipeline/#scripted-pipeline-fundamentals)

## Declarative Pipeline vs Scripted Pipeline

You can choose whether to use a declarative pipeline or a scripted pipeline.

# Time to start developing the pipeline!

At the beginning of this challenge, you were given instructions to spin up a Docker container. That container will only be used to access the Kubernetes cluster. To develop the pipeline, **you do not need the container**. Clone your forked Git repository somewhere in your **local machine** and use your favourite Text Editor / IDE to write the code.

**Clone your forked repository before starting the Stage 1 below**

## Pipeline Stage 1: Cloning the remote repository

Remember when you configured the pipeline (on the Jenkins UI) with a GitHub URL? That URL is used by Jenkins to only **get the Jenkinsfile and nothing else**. This means that once Jenkins grabs the Jenkinsfile from that repository, it will start executing the pipeline. At that point, you will not have access to the cloned repository contents (i.e. the application code). To double check that's true, modify the Jenkinsfile with the following content:

```
node() {
    echo "Your Pipeline works!"
    sh('ls -la')
}
```

Once the pipeline runs, the result of the `ls -la` command should be as follows:

```
[Pipeline] sh
+ ls -la
total 8
drwxr-xr-x 2 jenkins jenkins 4096 Feb 12 20:17 .
drwxr-xr-x 4 jenkins jenkins 4096 Feb 12 20:17 ..
```

As you can see, there are no files in the current directory. This means you will have to clone the repo **again**.

Check these links for more information about how to clone a repository and for pipeline examples:

[https://jenkins.io/doc/pipeline/steps/workflow-scm-step/](https://jenkins.io/doc/pipeline/steps/workflow-scm-step/)

[https://github.com/jenkinsci/pipeline-examples](https://github.com/jenkinsci/pipeline-examples)

**Note that the Pipeline: SCM Step plugin has already been installed for you.**

## Pipeline Stage 2: Running tests

The second stage of the pipeline would be to make sure that all tests are passing. In order to test the application in the repository you cloned in Stage 1, you will need to run the following commands **in your pipeline** (not locally):

```
npm install
npm test
```

**But beware!** This is when things will start getting confusing.

When you ran a pipeline like the following:

```
node() {
    echo 'This is a pipeline!'
}
```

Jenkins will spin up a Pod on Kubernetes with an executor container. The image name of the executor container is `jenkins/jnlp-slave:3.10-1`. If you run in your local machine:

```
docker run -it --rm jenkins/jnlp-slave:3.10-1 sh
```

Then, once you're in the container run:

```
$ npm
sh: 1: npm: not found
```

You will see npm is not installed. This is because the image `jenkins/jnlp-slave:3.10-1` does not have any language-specific tools installed. 

You have 2 options: you can either modify `jenkins/jnlp-slave:3.10-1`, install npm, generate a new image, push to a Docker repository and then use the new image in your pipeline (too much work, we don't want that), or you can leverage an image which already contains `npm`, like the `node:carbon-jessie` image, for example.

The second option is the most recommended one because you just reuse Docker images which are maintained by the community (just make sure it's a trustworthy one!) and you don't have to build and maintain images yourself. Now the question is, how can we tell Jenkins to run `npm install` and `npm test` inside another image (`node:carbon-jessie` instead of the default `jenkins/jnlp-slave:3.10-1`)?

Check the **Creating and Understanding Pipeline** section of [this article](https://akomljen.com/set-up-a-jenkins-ci-cd-pipeline-with-kubernetes/).

## Pipeline Stage 3: Build and push a Docker image to DockerHub

The last stage of this pipeline will be to deploy this application to Kubernetes. Therefore, before you do that you need to build a Docker image and publish it in a remote Docker repository (if you build the image locally on Jenkins and you're trying to deploy to another Kubernetes cluster, it won't work unless you publish this image to a public/private container registry). This is the stage where you will do that.

**In the Date Display application repository you will find a Dockerfile. You do not need to develop one yourself.**

## Pipeline Stage 4: Deploy to Kubernetes

If you've never worked with Kubernetes before, don't worry, we're going to explain in details how it works. If you've worked with Kubernetes before, we will not be providing any definition file. You can either develop one yourself or use `kubectl run` and specify the image you've just built (this application is so simple that it does not need any environment variables or volume mount). The next section will briefly explain how to deploy to Kubernetes. Skip it if you wish.

### Deploying to Kubernetes

The focus on this challenge is to build a CI/CD pipeline that would work in any environment, whether you're deploying applications to virtual machines, Kubernetes clusters etc. Therefore, this will be a very brief explanation how to deploy to Kubernetes. Check the [Kubernetes Dojo](https://github.com/thedojoseries/kubernetes-challenge) for an in-depth exercise on Kubernetes.

To deploy an application to Kubernetes, you will have to deploy a container (it's actually a Pod, but let's not get into too much details for now). If you've never worked with Kubernetes before, the command-line tool used to interact with the cluster is called `kubectl`. If you go to the container you span up in the Getting Started section, you can run commands like `kubectl get nodes` and you will get a response from the cluster where Jenkins is running.

To deploy a container, [take a look at this link](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/). You will not need to specify anything else than the image name.

### Good luck to all teams!

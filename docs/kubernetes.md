# Kubernetes Concepts

## **Kubernetes object model**

Each thing Kubernetes manages is represented by an **object**. We can view and change these objects, attributes, and state.

### The principle of **Declarative Management**:

-- tell it what you want the state of the objects, it will bring that state to there, and keep there.

--watch loop

**Kubernetes objects**:à Persistent entities representing the state of the cluster; its desired state and its current state…

`	`Various kinds of objects represent containerized applications, the **resources** that are available to them, and the **policies** that affect their behavior.

**Kubernetes objects** have two important elements. You give Kubernetes an **object spec** for each object you want to create. With this **spec**, you **define the desired state** of the object by providing the characteristics that you want.

The **object status** is simply the **current state of the object** provided by the Kubernetes control plane. By the way, we use the term Kubernetes control plane to refer to the various system processes that collaborate to make a Kubernetes cluster work.

![Timeline Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.001.png)

Each object is of a certain type or **'Kind'** as Kubernetes calls them.

**Pods** are the basic building block of the standard Kubernetes module, and they are the **smallest deployable Kubernetes object**. You may be surprised to hear me say that. Maybe you're expecting me to say the smallest Kubernetes object is the container. Not so.

**Every running container in a Kubernetes system is a Pod**. A Pod embodies the environment where containers live and that environment can accommodate **one or more containers**.

If there is more than one container in a Pod, they are tightly coupled and they **share resources** including **networking and storage**. ![Graphical user interface, application Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.002.png)

` `Kubernetes assigns **each Pod** a **unique IP address**. Every container within a Pod shares the network namespace including IP address and network ports. **Containers within the same Pod can communicate through localhost 127.0.0.1.**

A Pod can also specify a set of storage volumes to be shared amongst it's containers. By the way, later in this specialization, you'll learn how **Pods can share storage with one another** and not just a single Pod.

Let's consider a simple example, where you want three instances of the nginx web server, each in its own container, running all the time. How is this achieved in Kubernetes? Remember that Kubernetes embodies the principle of declarative management. You declare some objects to represent those nginx containers. What kind of object? Perhaps Pods! Now it is Kubernetes' job to launch those Pods and keep them in existence. But be careful, Pods are not self healing. If we want to keep all our nginx web servers not just in existence but also working together as a team, we might want to ask for them using a more sophisticated kind of object. We'll tell you more about this later in the module. Let's suppose we have given Kubernetes a desired state that consists of three nginx Pods, always kept running. We did this by telling Kubernetes to create and maintain one or more objects that represent them. Kubernetes compares the desired state to its current state.

![Diagram, timeline Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.003.png)

Let's imagine that our declaration of three nginx containers is completely new. The current state does not match the desired state, so Kubernetes, specifically it's control plane, will remedy the situation. Because the number of desired Pods running for the object we declared as three, and zero are presently running, three will be launched. The Kubernetes control plane will continuously monitor the state of the cluster, endlessly comparing reality to what has been declared and remedying state as needed. \(remedy—duzeltici onlem, tamir, care bulmak, geregine bakmak, onarmak, iyilestirmek, tedbir getirmek.)

## **Kubernetes Control Plane**

The Kubernetes **control plane**, which is the fleet of cooperating processes that make a kubernetes cluster work.

Even though you'll only work directly on a few of these components, it helps to know about them and the role each plays.

I'll build up a kubernetes cluster, part by part, explaining each piece as I go. After I'm done, I'll show you how a Kubernetes cluster running in GKE is a lot less work to manage the one you provision yourself. Okay, here we go.

First and foremost, your cluster needs computers. Nowadays, there are computers to compose your clusters. Usually our virtual machines. They always are in GKE but they could be physical computers, too. **One computer is** called the **control plane** and **the others are simply called nodes.**

The job of the **nodes** is **to run pods**.

The job of the **control plane** is **to coordinate the entire cluster**. We'll meet its control plane components first.

Several critical kubernetes components run under control plane. The single component that you **interact with directly** is called the **kube-APIserver**. This component's **job is to accept commands** that view or change the state of the cluster, including **launching pods**.

In the specialization, you will use the **kubectl** command frequently. This command's job is **to connect to the kubeAPIserver** and communicate with us using the kubernetes API.

**kubeAPIserver** also **authenticates incoming requests**, determines whether they are authorized and valid, and **manages admission control**. But it's not just the kubectl that talks with kube-APIserver.

In fact, **any query or change toward the cluster state must be addressed to the kubeAPIserver**.

**etcd** is the clusters database. Its job is to **reliably store the state of the cluster**. This includes all of the **cluster configuration data** and more dynamic information, such as what nodes are part of the cluster, what pods should be running and where they should be running. **You'll never interact directly with etcd**. I**nstead, kube-APIserver interacts with the database on behalf of the rest of the system**.

**kube-scheduler** is responsible **for scheduling pods onto nodes**. To do that, it evaluates the requirements of each individual pod and selecting which node is most suitable. But it doesn't do the work of actually launching the pods on the nodes. Instead, whenever it discovers a pod object that doesn't yet have an assignment to a node, it chooses a node and simply writes the name of that node into the pod object.

Another component of the system is responsible, then for launching the pods, and you'll see it very soon.

But how does kube-scheduleer decide where to run the pod? It knows the **state of all of the nodes**, and it will **obey constraints that you define** on where a pod may run. Based on hardware, software and policy. For example, you might specify a certain pod is only allowed to run on nodes with a certain amount of memory. You can also define **affinity specifications**, which calls groups of pods to prefer running on the same node. Or **anti affinity specifications**, which ensure that pods do not run on the same node. You will learn more about some of these tools in later modules.

**kube-controller-manager** has a broader job. It **continuously monitors** the **state of the cluster** through **kube-APIserver**. Whenever the current state of the cluster doesn't match the desired state, **kube-controller-manager** will attempt to make changes **to achieve the desired state**. It's called the controller manager because many kubernetes objects are managed by **loops of code called controllers**. These loops of code handle the process of remediation. **Controllers** will be very useful for you. To be specific, you'll learn to use certain kinds of kubernetes controllers to manage workloads. For example, remember our problem of keeping three engine x pods always running.

We can gather them together into a **controller object** called a **deployment**. That not only keeps them running, but also let's us scale them and bring them together under the front end. We'll meet deployments later in this module.

Other kinds of controllers have system level responsibilities. For example, **node controller**'s job is **to monitor and respond when a node is offline.**

**kube-cloud-manager** manages controllers that interact with the underlying cloud providers. For example, if you manually launch a kubernetes cluster on Google Compute Engine, **kube-cloud-manager** would be responsible for bringing in Google cloud features like **load balancers** and **storage volumes** when you needed them.

Each node runs a small family of control plane components too. For example, each node runs a kubelet. You can think of a **Kubelet** as **kubernetes agent on each node**. When the **kube-APIserver** wants to start a pod on a node, it **connects** to that **node's** **kubelet**. Kubelet uses the **container runtime** to start the pod and monitors its life cycle. Including **readiness** and **liveliness** probes and **reports back to kube-APIserver**. Do you remember our use of the term **container runtime** in the previous module? This **is the software that knows how to launch a container from a container image.**

` `![Diagram Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.004.png)

The world of kubernetes offers several choices for container runtimes. But the Linux distribution that GKE uses for its node launches containers using **Container D. >> The runtime component of Docker**.

**Kube-proxy**’s job is to **maintain the network connectivity** among the pods in a cluster. In **open source kubernetes**, it does so using the **firewall capabilities of IP tables**, which are **built into the Linux kernel**. Later, in the specialization, we will learn how GKE handles part networking.

## **Kubernetes Object Management**

All **Kubernetes objects** are **identified** by a **unique name** and a **unique identifier.**

Let's return once again to our example in which we want three nginx web servers running all the time. Well, the simplest way would be to declare three Pod objects and specify their state: for each, a Pod must be created and an nginx container image must be used. Let's see how we declare this.

**You define the objects** you want Kubernetes to create and maintain with **manifest files**. These are ordinary text files. You may write them in **YAML or JSON format**. YAML is more human-readable and less tedious to edit, and we will use it throughout this specialization. ![Table

Description automatically generated with low confidence](./images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.005.png)This YAML file defines a desired state for a Pod: its name and a specific container image for it to run.

Your manifest files have certain required fields.

**'apiVersion'** describes which Kubernetes Api version is used to create the object. The Kubernetes protocol is versioned so as to help maintain backwards compatibility.

**'Kind'** defines the **object** you want, in this case a Pod,

and **'metadata'** helps identify the object using **name**, **unique** **ID**, and an optional **namespace**. You can define **several related objects in the same YAML file** and it is a best practice to do so. One file is often easier to manage than several. Another even more important tip: You should save your YAML files in version-controlled repositories. This practice makes it easier to track and manage changes and to back-out those changes when necessary. It's also a big help when you need to recreate or restore a cluster. Many GCP customers use Cloud Source Repositories for this purpose, because that service lets them control the permissions of those files in the same way as their other GCP resources.

When you create a Kubernetes object, you name it with a string. **Names must be unique.** ![Diagram Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.006.png)**Only** **one object of a particular kind can have a particular name at the same time in a Kubernetes namespace**. However, if an object is deleted, its name can be reused. Alphanumeric characters, hyphens, and periods are allowed in the names, with a maximum character length of 253.

Each object generated throughout the life of a cluster has a unique ID generated by Kubernetes. This means that no two objects will have the same **UID** throughout the life of a cluster.

**'Labels'** are **key-value pairs** with which you **tag** your objects during or after their creation. ![Text Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.007.png)Labels help you **identify and organize objects** and subsets of objects. For example, you could create a label called 'app' and give as its value the application of which this object is a part.

In this simple example, a Deployment object is labeled with three different key values: its application, its environment, and which stack it forms a part of. Various contexts offer ways to select Kubernetes resources by their labels. In this specialization, you will spend plenty of time with the kubectl command. Here's an example of using it to show all the Pods that contain a label called 'app' with a value of 'nginx'. Label selectors are very expressive. You can ask for all the resources that have a certain value for a label, all those that don't have a certain value, or even all those that have a value in a set you supply.

So one way to bring three nginx web servers into being would be to declare three Pod objects, each with its own section of YAML. Kubernetes' default scheduling algorithm prefers to **spread the workload evenly across the nodes available** to it. So we'd get a situation like this one. ![Diagram, timeline Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.008.png)Looks good, doesn't it? Maybe not. Suppose I want 200 more nginx instances. Managing 200 more sections of YAML sounds very inconvenient. Here's another problem. **Pods don't heal or repair themselves**, and they're not meant to run forever. ![Diagram Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.009.png)They are designed to be **ephemeral** and **disposable**. For these reasons, there are better ways to manage what you run in Kubernetes than specifying individual Pods. You need a setup like this to maintain an application's high availability along with horizontal scaling. So, how do you tell Kubernetes to maintain the desired state of three nginx containers?

We can instead **declare a controller object** whose job is **to manage the state of the Pods**. ![Table Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.010.png)Some examples of these objects: **Deployments, StatefulSets, DaemonSets**, and **Jobs**. We'll meet all of these in our specialization.

**Deployments** are a great choice for long-lived software components like web servers, especially when we want to manage them as a group. In our example, when **kube-scheduler** schedules Pods for a Deployment, it notifies the **kube-APIserver**.

` `![Graphical user interface Description automatically generated with low confidence](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.011.png)

These changes are constantly monitored by **controllers**, especially by the **Deployment controller**. The Deployment controller will monitor and maintain three nginx Pods. If one of those Pods fails, the Deployment controller will recognize the difference between the current state and the desired state, and will try to fix it by launching a new Pod. Instead of using multiple YAML manifests or files for each Pod, you used a single Deployment YAML to launch three replicas of the same container. **A Deployment ensures that a defined set of Pods is running at any given time**. ![Table Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.012.png)**Within** its object **spec**, you specify how many replica Pods you want, how Pods should run, which containers should run within these Pods, and which Volumes should be mounted. Based on these templates, controllers maintain the Pod's desired state within a cluster. Deployments can also do a lot more than this, which you will see later in the course.

When Kubernetes schedules a Pod, it’s important that the containers have enough resources to actually run. If you schedule a large application on a node with limited resources, it is possible for the node to run out of memory or CPU resources and for things to stop working!

It’s also possible for applications to take up more resources than they should. This could be caused by a team spinning up more replicas than they need, possibly to artificially decrease latency, or to a bad configuration change that causes a program to go out of control and use 100% of the available CPU. Regardless of whether the issue is caused by a bad developer, bad code, or bad luck, **what’s important is that you are in control.**

When you specify a Pod, you can optionally specify how much of each resource a container needs. The most common resources to specify are CPU and memory (RAM); however, there are others. Let's take a look at the mechanisms Kubernetes uses to control these resources.

So how do you keep everybody's work on your cluster tidy and organized? **Kubernetes allows you to abstract a single physical cluster into multiple clusters known as 'namespaces'.** **Namespaces** provide scope for naming resources such as Pods, Deployments, and controllers.

As you can see in this example, there are three namespaces in this cluster: Test, Stage, and Prod. Remember that you cannot have duplicate object names in the same namespace. You can create three Pods with the same name, nginx in this case, but only if they don't share the same namespace.

` `![A picture containing timeline Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.013.png)

If you attempt to create another Pod with the same name 'nginx Pod' in namespace 'test', you won't be allowed. Object names need only be unique within a namespace, not across all namespaces.

**Namespaces** also let you implement **resource quotas** across the cluster. These quotas define limits for resource consumption within a namespace. They're not the same as your GCP quotas, which we discussed in an earlier module. These quotas apply specifically to the Kubernetes cluster they're defined on. You're not required to use namespaces for your day-to-day management. You can also use **labels**. Still, namespaces are a valuable tool. Suppose you want to **spin up a copy of a deployment** as a **quick test**. Doing so in a **new namespace makes it easy and free of name collisions**.

There are **three initial namespaces** in a cluster. The first is a **default namespace**, for objects with no other namespace defined. Your workload resources will use this namespace by default.

Then there is the **kube-system** **namespace** for objects created by the Kubernetes system itself. We'll see more of the object kinds in this diagram elsewhere in this specialization. When you use the **kubectl** command, **by default**, items in the **kube-system namespace are excluded**, but **you can choose to view its contents explicitly**.

The third namespace is **the kube-public namespace** for objects that are publicly readable to all users. **kube-public is a tool for disseminating information** to everything running in a cluster. You're not required to use it, but it can come in handy, especially when everything running in a cluster is related to the same goal and needs information in common.

![A picture containing table Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.014.png)

**You can apply a resource to a namespace when creating it using a command-line namespace flag**. **Or, you can specify a namespace in the YAML file for the resource**. **Whenever possible, apply namespaces at the command line level.**

![Graphical user interface, text, application Description automatically generated](images/Aspose.Words.9e3d91d3-f65a-4442-99e7-3d70c6efd835.015.png)

This practice makes your YAML files more flexible. For example, some day you might want to create two completely independent instances of one of your deployments, each in its own namespace. This is difficult if you have chosen to embed namespace names in your YAML files.

**A note about Deployments and ReplicaSets**

The example nginx Deployment you saw in the previous video was simplified. In practice, you would launch a Deployment object to manage your desired three nginx Pods, just as the video described. Note, though, that the Deployment object would create a ReplicaSet object to manage the Pods. The diagram in the video leaves out this detail. You will work with **Deployment objects directly much more often** than ReplicaSet objects. But it's still helpful to know about ReplicaSets, so that you can better understand how Deployments work. For example, one capability of a Deployment is to allow a rolling upgrade of the Pods it manages. To perform the upgrade, the Deployment object will create a second ReplicaSet object, and then increase the number of (upgraded) Pods in the second ReplicaSet while it decreases the number in the first ReplicaSet.

**A note about Services**

Services provide **load-balanced access** to specified Pods. There are three primary types of Services:

● **ClusterIP**: Exposes the service on an IP address that is _only accessible from within this cluster_. This is the default type.

● **NodePort**: Exposes the service on the IP address of each node in the cluster, at a specific port number.

● **LoadBalancer**: Exposes the service **externally**, using a load balancing service provided by a cloud provider.

In Google Kubernetes Engine, LoadBalancers give you access to a regional Network Load Balancing configuration by default. To get access to a global HTTP(S) Load Balancing configuration, you can use an **Ingress** object.

You will learn more about Services and Ingress objects in a later module in this specialization.

**Controller objects to know about**

This reading explains the relationship among several Kubernetes controller objects:

● ReplicaSets

● Deployments

● Replication Controllers

● StatefulSets

● DaemonSets

● Jobs

A **ReplicaSet** **controller** ensures that a population of Pods, all identical to one another, arevrunning at the same time.

Deployments let you do declarative updates to ReplicaSets and Pods.

In fact, Deployments manage their own ReplicaSets to achieve the declarative goals you prescribe, so you will **most commonly work with Deployment objects**.

Deployments let you create, update, roll back, and scale Pods, using ReplicaSets as needed todo so. For example, when you perform a rolling upgrade of a Deployment, the Deployment object creates a second ReplicaSet, and then increases the number of Pods in the new ReplicaSet as it decreases the number of Pods in its original ReplicaSet.

**Replication Controllers** perform a similar role to the combination of ReplicaSets and Deployments, but their use is no longer recommended. Because Deployments provide a helpful "front end" to ReplicaSets, this training course chiefly focuses on Deployments.

If you need to deploy applications that maintain local state, **StatefulSet** is a better option. A StatefulSet is similar to a Deployment in that the Pods use the same container spec. The Pods created through Deployment are not given persistent identities, however; by contrast, **Pods created using StatefulSet have unique persistent identities with stable network identity and persistent disk storage.**

If you need **to run certain Pods on all the nodes within the cluster or on a selection of nodes, use DaemonSet**. DaemonSet ensures that a specific Pod is always running on all or some subset of the nodes. If new nodes are added, DaemonSet will automatically set up Pods in those nodes with the required specification. The word "daemon" is a computer science term meaning a **non-interactive process** that provides useful services to other processes. A Kubernetes cluster might use a DaemonSet to ensure that a logging agent like fluentd is running on all nodes in the cluster.

The **Job** controller creates one or more Pods required to run a task. When the task is completed, Job will then terminate all those Pods. A related controller is **CronJob**, which runs Pods on a time-based schedule.

Later modules in this specialization will cover these controllers in more depth.

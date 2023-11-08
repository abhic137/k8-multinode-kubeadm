# Instructions to run the test deployment.
Node-to-node communication in a Kubernetes cluster typically involves the pods running on different nodes communicating with each other. Let's create an example that demonstrates node-to-node communication. We'll deploy a simple client-server application where the client pod, running on one of the worker nodes, communicates with the server pod, running on a different worker node.
## Run all these commands in the master node.

1. Create a Server Deployment and Service:
   First, create a deployment for the server application and expose it using a service to make it accessible to other nodes. Here's an example YAML for the server:

   ```yaml
   # server-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: server-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: server
     template:
       metadata:
         labels:
           app: server
       spec:
         containers:
         - name: server
           image: nginx:latest
   ```

   Create the deployment:

   ```
   kubectl apply -f server-deployment.yaml
   ```

   Next, create a service to expose the server deployment:

   ```yaml
   # server-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: server-service
   spec:
     selector:
       app: server
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
     type: NodePort
   ```

   Create the service:

   ```
   kubectl apply -f server-service.yaml
   ```

2. Create a Client Pod:
   Now, create a client pod that will communicate with the server pods. Here's an example YAML for the client:

   ```yaml
   # client-pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: client-pod
   spec:
     containers:
     - name: client
       image: busybox:latest
       command: ["sh", "-c", "while true; do wget -qO- server-service; sleep 5; done"]
   ```

   Create the client pod:

   ```
   kubectl apply -f client-pod.yaml
   ```

3. Observe Node-to-Node Communication:
   The client pod is continuously making HTTP requests to the server pods through the "server-service." These requests will go through the network, showcasing node-to-node communication. You can observe this communication by checking the logs of the client pod:

   ```
   kubectl logs client-pod
   ```

   You'll see that the client is able to communicate with the server pods, which are running on different nodes in your Kubernetes cluster.

This example demonstrates how pods on different nodes in your Kubernetes cluster can communicate with each other using services and network routing.

# Verification 
You can check and verify if the communication is happening in your Kubernetes cluster by examining the logs and the status of your pods. Here are some steps to verify the communication:

1. Check the Client Pod Logs:
   To see if the client pod is successfully communicating with the server pod, you can check the logs of the client pod using the following command:

   ```bash
   kubectl logs client-pod
   ```

   This command will display the output of the client pod, showing if it's able to connect to the server and retrieve data.

2. Check the Server Pod Logs:
   You can also check the logs of the server pods to see if they are receiving requests. Use the following command to view the logs of one of the server pods:

   ```bash
   kubectl logs <server-pod-name>
   ```

   Replace `<server-pod-name>` with the name of one of your server pods. The logs should show that the server received requests.

3. Verify Service Endpoints:
   You can verify that the client is able to reach the server by checking the endpoints of the service. Run the following command to see if the server service has endpoints:

   ```bash
   kubectl describe service server-service
   ```

   Look for the "Endpoints" section, which should list the IP addresses of the server pods. If the endpoints are populated, it means the service is correctly directing traffic to the server pods.

4. Check Pod Status:
   You can check the status of your client and server pods to ensure they are in the "Running" state. Use the following command to get the status of your pods:

   ```bash
   kubectl get pods
   ```

   If any pod is in a state other than "Running," it may indicate a problem with the pod's communication.

5. Use Network Tools:
   You can use network tools like `curl` or `wget` from within the client pod to directly test the connectivity to the server. For example:

   ```bash
   kubectl exec -it client-pod -- sh
   # Inside the client pod, you can run:
   wget -qO- server-service
   ```

   This command inside the client pod should return the content from the server if the communication is successful.

By following these steps, you can verify that the communication between your client and server pods is working as expected in your Kubernetes cluster.

# Simple Explanation
In a simpler way:

Imagine you have three computers: one is the "boss" (the master node), and the other two are "workers" (worker nodes). The boss tells the workers what to do, and they help complete tasks.

In this example, we have a "server" computer (server pod) and a "client" computer (client pod).

1. The boss (master node) tells one worker (worker1) to become the server and serve web pages.

2. The boss also tells another worker (worker2) to become the client and keep asking the server for web pages.

3. The client worker keeps sending requests to the server worker, asking for web pages. This is like you using a web browser to visit different websites.

4. Even though the client and server are on different worker computers, they can talk to each other. This is what we call "node-to-node communication."

5. The boss (master node) can see what's happening and helps coordinate the communication between the client and server.

So, the example is like having two computers (workers) in your class. One computer shows a picture (server), and the other computer keeps asking for more pictures (client), and they can share pictures even though they're not sitting next to each other. The teacher (master node) watches over to make sure everything goes smoothly.

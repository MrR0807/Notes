## Namespaces

<details><summary>Get namespace</summary>

```
kubectl get namespace
```

</details>


<details><summary>Get pods in 'hello' namespace</summary>

```
kubectl get pods -n=hello
```

</details>


<details><summary>Set current namespace to hello</summary>

```
kubectl config set-context --current --namespace=hello
```

</details>

<details><summary>Create goodbye namespace</summary>

```
kubectl create namespace goodbye
```

</details>

<details><summary>Delete goodbye namespace</summary>

```
kubectl delete namespaces goodbye
```

</details>

<details><summary>List all pods in your cluster</summary>

```
kubectl get pods --all-namespaces
```

</details>

<details><summary>Add label color=red to a pod named foo</summary>

```
kubectl label pods foo color=red
```

</details>

<details><summary>Remove label color from pod foo</summary>

```
kubectl label pods foo color-
```

</details>

<details><summary>Create a deployment with label color=red, app=hello</summary>

```
kubectl create deployment foo --image=bluebox --labels="color=red,app=hello"
```

</details>

<details><summary>Apply a label "canary=true" to a running deployment name foo</summary>

```
kubectl label deployments foo "canary=true"
```

</details>

<details><summary>Show deployments with labels </summary>

```
kubectl get deployments -L canary
```

</details>

<details><summary>        </summary>

```

```

</details>

<details><summary>        </summary>

```

```

</details>

<details><summary>        </summary>

```

```

</details>

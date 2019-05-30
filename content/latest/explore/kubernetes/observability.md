<h2 id="kubernetes-create-universe">1. Create universe</h2>

If you have a previously running local universe, destroy it using the following.

```sh
$ kubectl delete -f yugabyte-statefulset.yaml
```

Start a new local cluster - by default, this will create a 3 node universe with a replication factor of 3.

```sh
$ kubectl apply -f yugabyte-statefulset.yaml
```

<h2 id="kubernetes-clean-up">Clean up (optional)</h2>

Optionally, you can shutdown the local cluster created in Step 1.

```sh
$ kubectl delete -f yugabyte-statefulset.yaml
```

Further, to destroy the persistent volume claims (**you will lose all the data if you do this**), run:

```sh
kubectl delete pvc -l app=yb-master
kubectl delete pvc -l app=yb-tserver
```

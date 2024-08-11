new-alias kb kubectl

>kb apply -f .\lab\deployments.yaml

> kb get pods
NAME                                 READY   STATUS    RESTARTS   AGE
lab-numbers-api-8d86cd779-zjcdr      1/1     Running   0          48s
lab-numbers-web-79755bb4b5-vbkgg     1/1     Running   0          48s
lab-numbers-web-v2-c7bf58c96-fnfxd   1/1     Running   0          48s

> kb get svc 
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   89s


```
apiVersion: v1
kind: Service
metadata:
  name: numbers-api
spec:
  selector:
    app: numbers-api
  type: ClusterIP
  ports:
    - port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: numbers-web-v2
spec:
  selector:
    app: numbers-web-v2
  type: LoadBalancer
  ports:
    - port: 8088
	  targetPort: 80
```



# Scalabilità  e Resilienza in Kubernetes: come le app Cloud-Native si auto adattano al carico e rinascono dalle ceneri

## Premessa

Questa Demo è volta a dimostrare la funzionalità dell'`horizontal pod autoscaler` e della `failure recovery` dei cluster Kubernetes (K8s).

## Cosa faremo

Creeremo un cluster Kubernetes con `apache` http server e lo porremo sotto stress per testare le funzionalità di adattamento di Kubernetes al traffico rete ed eventuali crash dell'applicazione.

## Creazione del cluster

Di seguito le alternative: usiamo un cloud provider oppure in locale con k3d/minikube.

### Creazione del cluster su un vero Cloud Provider (Civo)
Usiamo la CLI:
```bash
~  $ civo kubernetes create linuxday --cluster-type=talos --nodes=2 --region=lon1
The cluster linuxday (67d7e831-579a-4185-abe4-689ac698199f) has been created
```
Recuperiamo il `kubeconfig` e puntiamo al cluster:
```bash
~  $ civo kubernetes config linuxday --region=lon1 > ~/.kube/config_linuxday
~  $ export KUBECONFIG=~/.kube/config_linuxday
```
Aggiungiamo un alias per comodità e testiamo la connessione al cluster:
```bash
~  $ alias k='kubectl'
~  $ k get ns
NAME              STATUS   AGE
default           Active   2m26s
kube-node-lease   Active   2m26s
kube-public       Active   2m26s
kube-system       Active   2m26s
```

## Installazione del `metrics-server`

Installiamo il metrics-server, indispensabile per il funzionamento dell'horizontal pod autoscaler:
```bash
~  $ k apply -f metrics-server.yaml
```

## Installazione dei componenti

Installiamo tutti i componenti:
```bash
~  $ k apply -f php-apache.yaml
```

## Test del `horizontal-pod-autoscaler`

Osserviamo (su due terminali diversi) in tempo reale i pod e l'horizontalpodautoscaler:
```bash
# terminale 1
~  $ k get pods --watch
# terminale 2
~  $ k get hpa --watch
```
Poi avviamo il carico scegliendo uno dei seguenti modi:

Utilizzando l'IP del Loadbalancer (se avviato tramite il Cloud Provider):
```bash
~  $ LB_IP=$(k get svc/php-apache-svc -o jsonpath={.status.loadBalancer.ingress[0].ip})
~  $ while sleep 0.01; do curl http://$LB_IP:80>/dev/null; done
```
utilizzando il `kubectl port-forward`:
```bash
~  $ k port-forward svc/php-apache-svc 8888:80
# su un altro terminale
~  $ while sleep 0.01; do curl http://localhost:8888>/dev/null; done 
```

utilizzando un pod nel cluster creato ad-hoc per effettuare le chiamate alla nostra app:
```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache-svc; done"
```

## Test di recovery
TODO

Proviamo ad cancellare il pod
```bash
~  $ k delete pods/...
```
... il replicaset


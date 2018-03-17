# armazenamento_distribuido_Cloud-Native_rook.io-ceph

Neste repositório é descrita a configuração e instalação do [ROOK](https://rook.io/), um serviço de armazenamento distribuído para ambientes *Cloud-Native*, em cima de nosso [cluster kubernetes](https://github.com/ctic-sje-ifsc/baremetal_rancherOS_rke_kubernetes). 

O primeiro passo é implantar os componentes do sistema Rook, que incluem o agente Rook executado em cada nó no seu cluster, bem como o pod do Rook *operator*:

```$ kubectl create -f rook-operator.yaml```  

Verificar o *status* dos *pods* até que todos estejam em *Running*:

```$ kubectl -n rook-system get pod```  

Agora que o Rook *operator* e Rook *agent* estão em execução, é possível criar o *cluster* Rook:  

```$ kubectl create -f rook-cluster.yaml```

Verificar se os pods rook-ceph-ods-XX, rook-ceph-ceph-monX, rook-ceph-mgr0 e rook-api estão em *Running*:  

```$ kubectl -n rook get pod```  
	
Criar o Rook Toolbox:

```$ kubectl create -f rook-tools.yaml```  
```$ kubectl -n rook get pod rook-tools```

Acessar o toolbox:  

```$ kubectl -n rook exec -it rook-tools bash```

Criar o Block Storage/Storage Class:

```$ kubectl create -f rook-storageclass.yaml```  


## Consumindo um armazenamento persistente criando o PVC.

Baseado em [Rook.io distributed storage for Kubernetes](http://sonamhava.blogspot.com.br/2017/05/rookio-distributed-storage-for.html).

Exemplo de PCV:

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvclaim-0
  annotations:
    volume.beta.kubernetes.io/storage-class: srv-pool
  labels:
    app: example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Descobrir o nome do PV criado:

```$ kubectl get pv```

Mudar o ReclaimPolicy para Retain do PV criado:

```$ kubectl patch pv -p '{"spec": {"persistentVolumeReclaimPolicy":"Retain"}}' pvc-XXXXX...```


Consumir o PVC criado utilizando o nome, por exemplo nesse caso:

* No values.yml de um [helm charts](https://github.com/kubernetes/charts): 
```yml
 existingClaim: pvclaim-0
 ```

* Ou em um *deploy/pod*:
[...]
```yml
     volumes:
      - name: data
        persistentVolumeClaim:
          claimName: pvclaim-0
```
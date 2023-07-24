- kube-apiserver (api) –> K8s’in beyni, ana haberleşme merkezi, giriş noktasıdır. Bir nev-i Gateway diyebiliriz. Tüm componentler ve node’lar, kube-apiserver üzerinden   iletişim kurar. Ayrıca, dış dünya ile platform arasındaki iletişimi de kube-apiserver sağlar. Bu denli herkesle iletişim kurabilen tek componenttir. Authentication ve  Authorization görevini üstlenir.  
- etcd -> K8s’in tüm cluster verisi, metada bilgileri ve Kubernetes platformunda oluşturulan componentlerin ve objelerin bilgileri burada tutulur. Bir nevi Arşiv odası.   etcd, key-value şeklinde dataları tutar. Diğer componentler, etdc ile direkt haberleşemezler. Bu iletişimi, kube-apiserver aracılığıyla yaparlar.  
kube-scheduler (sched) -> K8s’i çalışma planlamasının yapıldığı yer. Yeni oluşturulan ya da bir node ataması yapılmamış Pod’ları izler ve üzerinde çalışacakları bir   node seçer. (Pod = container) Bu seçimi yaparken, CPU, Ram vb. çeşitli parametreleri değerlendirir ve bir seçme algoritması sayesinde pod için en uygun node’un   hangisi olduğuna karar verir.  
- kube-controller-manager (c-m) -> K8s’in kontrollerinin yapıldığı yapıdır. Mevcut durum ile istenilen durum arasında fark olup olmadığını denetler. Örneğin; siz 3 cluster istediniz ve k8s bunu gerçekleştirdi. Fakat bir sorun oldu ve 2 container kaldı. kube-controller, burada devreye girer ve hemen bir cluster daha ayağa kaldırır. Tek bir binary olarak derlense de içerisinde bir çok controller barındırır:

- Api Server = Tüm komponentlerin haberleşmesini sağlayan bir merkezdir.  
- etcd = tüm cluster metada verilerinin key value seşklinde tutuldugu yerdir.
- kube-scheduler = oluşturulacak podun belirli secme algoritmalarına göre ayaga kalkacagı node u secer. Bunu pod.yml a yazar.
- kube controller manager = kontrol merkezi olarak düşünülebilir. .yaml dosyasında oluşturdugumuz podda örneğin node uzerinde 3 tane calıssın istiyoruz bir tanesi pod  
kapandı bu kısımda cm devreye girer ve podu ayaga kaldırır.
- kubelet = ETCD yi izleyerek kube-scheduler ın algoritmalarına göre seçtiği node da containerd ye istek göndererek imajı oluşturur.  
- kube-proxy = podlar uzeridenki network kurallarını ve trafik akışını yönetir.  
![image](https://user-images.githubusercontent.com/73287349/227900843-23c7940b-449d-496a-8f3e-7cb6bf808cb7.png)  
![image](https://user-images.githubusercontent.com/73287349/227901136-b35a853a-2cab-4fd4-844e-7fd86cf8bd85.png)  


## Pod
- Kubernetes container orkestra şefi. K8s de containerlar podlar üzerinde oluşturulur.Podlar içerisinde 1 veya birden fazla container olablir.(Best practice 1P = 1C)  
- pod calışma süreci = Her pod’un unique bir ID’si (uid) vardır ve unique bir IP’si vardır. Api-server, bu uid ve IP’yi etcd’ye kaydeder. Scheduler ise herhangi bir podun node ile ilişkisi kurulmadığını görürse, o podu çalıştırması için uygun bir worker node seçer ve bu bilgiyi pod tanımına ekler. Pod içerisinde çalışan kubelet servisi bu pod tanımını görür ve ilgili container’ı çalıştırır.  
- Aynı pod içerisindeki containerlar aynı node üzerinde çalıştırılır ve bu containerlar arasında network izolasyonu yoktur. Yani localhost üzerinden biribirleriyle hbaerleşir.  

#### Komutlar
``` kubectl run <podname> --image=nginx --port=80 --labels="app=fe" ```  
```kubectl get pods -o wide```
```kubectl delete pod <podname>```  
```kubectl describe pod <podname>```  
```kubectl logs (-f canlı bağlanır)<podname>```  
```kubectl exec -it first-pod -- /bin/sh```  
```kubectl exec -it first-pod -c <containername> -- /bin/sh``` 
- .yaml uzantılı dosya ile pod oluşturulubalir. "---" birden fazla pod tanımlanabilir.  
- apiversion -> Oluşturmak istediğimiz object’in hangi API üzerinde ya da endpoint üzerinde sunulduğunu gösterir.kubectl explain pod diyerek hangi api versionu koştugunu ögrenebiliriz.  
- kind –> Hangi object türünü oluşturmak istiyorsak buraya yazarız. ÖR: pod  
- metadata -> Object ile ilgili unique bilgileri tanımladığımız yerdir. ÖR: namespace, annotation vb.  
- spec -> Oluşturmak istediğimiz object’in özelliklerini belirttiğimiz yerdir. Her object için gireceğimiz bilgiler farklıdır. Burada yazacağımız tanımları, dokümantasyondan bakabiliriz.  
- oluşturulan yaml dosyasını çaliştırma : ```kubectl apply -f <...yml>```  
- oluşturulan yaml dosyasını silme : ```kubectl delete -f <...yml>```  
- oluşturulan yaml dosyasını editleme : ```kubectl edit -f <...yml>```  
örnek bir pod.yml:  
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
### pod yaşam döngüsü
- Pending : kubectl apply -f pod.yml şeklinde pod oluşturdugumuzda ilk olarak pending sürecine girer ve obje bilgileri metadata ya kaydedilir. Bu aşamada Kube scheduler tarafından herhangi bir node seçilmemiştir.  
- Creating : Kube scheduler tarafından pending aşamnasında hazırlanıp bir node ataması bekleyen objeye bu süreçte node ataması yapılır.  
- imagePullbackof : Bu aşamada kubelet bir containerd sayesinde registry de kayıtlı bir imajı belirlenen node üzerinde çalıştırmaya çalışır.Eğerki image da bir hata olursa örneğin image name yanlış yazılırsa kubelet bu süreçte tekrar tekrar registryden image ı çekmeye çalışır.  
Container uygulamasının durmasına karşılık, Pod içerisinde bir RestartPolicy tanımlanır ve 3 değer alır:  
- Always -> Kubelet bu container’ı yeniden başlatır.  
- Never -> Kubelet bu container’ı yeniden başlatmaz  
- On-failure -> Kubelet sadece container hata alınca başlatır.  
..  
- Succeeded -> Pod başarıyla oluşturulmuşsa bu duruma geçer.  
- Failed -> Pod başarıyla oluşturulmamışsa bu duruma geçer.  
- Completed -> Pod başarıyla oluşturulup, çalıştırılır ve hatasız kapanırsa bu duruma geçer.  
-  CrashLookBackOff -> Pod oluşturulup sık sık kapanıyorsa ve RestartPolicy’den dolayı sürekli yeniden başlatılmaya çalışılıyorsa, k8s bunu algılar ve bu podu bu state’e getirir. Bu state de olan podlar belirli aralıklarla oluşturulmaya çalışılıur.  
- Bir pod oluşturulacakken önce etdye kaydedilir(Pending).Daha sonra podun üzerinde koşacağı node kube-schduler tarafından ayarlanır ve etcd ye kaydedilir(Creating).Daha sonra etcd kayıtlı olan bilgileri izleyen kubelet containerd ile imajları oluşsturuur. 
### Labels
- Pod üzeriden tanımlanan label lar gruplandırma özelliği sayesinde filtreleme işlemlerinde ve pod selector ile pod seçme kısmında kullanılır.  
```kubectl get pods -l "app=first"```  
```kubectl get pods -l "!app"```  
```kubectl label pods pod1 app=front-end```ekleme    
```kubectl label pods pod1 app-```silme  
Kubescheduler tarafından otomatik bir şekilde node ataması yapılır. Bunu manuel hale getiirmek için nodeSelector: hddtype: ssd , nodeName: worker1 şeklinde node seçimi yapılabilir.  
### Annotation
metada altında annotations:.. şeklinde podu oluşturan bilgisi vs. şeklinde podun çalıması ile pek de alakalı olamayan ama aynı labellar gibi özellik gösteren bir belirteçtir.Örnek bir senaryoda şu podu oluştuan kullanıcıya mail at tarzı işlerde kullanılabilir.  
### Init container
Pod içerisined ana containerın çalışması  daha başlamadan çalışacak containerı belirtiriz.   
### Namespace
Namespace aynı package ler gibi diğer diğer ortamlaradan izole bir şekilde mgurplandırma özelliği sağlar.  
```kubectl get pods -n <namespaceName>```  
```kubectl get pods -A```  
```kubectl create namespace new```  
````kubectl config set-context --current --namespace=<namespaceName>````  
## Deployment
kubernetes de genellikle singleton podlar yaratılmaz.Podları bir üst seviye obje olan deployments ile yönetiriz. Deployment ile farklı farklı çalışması için farklı farklı node lara dağılan podlar deployments ile orkestre edilir. İstanilen sayı kadar pod oluşturma bunların bakımı kontrolü deployments tarafından sağklanır.  
deployments objesinde spec altında bir template içerisinde container oluşturabilirz.Örnek bir deployment yaml:
``` 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
replica sayısı podun kaç replicasi oldugunu belirtir.Selector altında match labels ile podun labelı ile eşleme yapıyoruz.  
bunun harici impreative şekilde de deployment oluştutrulabilir.  
``` kubectl create deployment <deploymentName> --image=nginx:latest --replicas=2 ```  
podların imageları değiştirebiliriz.  
```kubectl set image deployment/firstdeployment nginx=httpd```  
scale edebilriz.  
```kubectl scale deployment/nginx-deployment --replicas=10```  
deployment da podların güncellemelerini yönetme:  
```
strategy:
  type:recreate
```
- Bu şekilde pod güncelleme strategy si belirtildiğinde tüm podlar ölerek en baştan create edilir.  
```
strategy:  
  type: RollinUpdate  
  rollingUpdate:
    maxUnavailable:2 (en fazla bu sayıda podu sil)
    maxSurge: 2 (10 replicam varsa en fazla 12 yap öyle güncellemeye devam et)
```
- bu şekilde pod güncelleme strategy si belirtildiğinde belirlenen özelliklere göre şekillenir.  
--record flag i ile değişiklik version olarak kaydedilir.  
yapılan güncellemnin durumunu görmek için:
```kubectl rollout status deployment rolldeployment```  
yapılan güncelklemelerin listelenmesi için:
```kubectl rollout history deployment rolldeployment ```  
yapılan güncellemelerde farklı revisona geçmk için:  
```kubectl rollout undo deployment rolldeployment --to-revision=1```  
## Healthcheck
- liveneesprobe :  pod un çalışma zamanında sağlık durumunu kontrol etmek için kkullanılan object tyepe dir.
```
livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
```
livenessProbe:
       httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```
```
tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```
- readinees Probe = Bazen uygulamalar geçici olarak trafiğe hizmet veremez. Örneğin, bir uygulamanın başlatma sırasında büyük veri veya yapılandırma dosyaları yüklemesi veya başlatma sonrasında harici hizmetlere bağlı olması gerekebilir. Bu gibi durumlarda, uygulamayı öldürmek istemezsiniz, ancak ona istek göndermek de istemezsiniz.
```
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```
```
readinessProbe:
          httpGet:
            path: /ready	# Bu endpoint'e istek atılır, OK dönerse uygulama çalıştırılır.
            port: 80
          initialDelaySeconds: 20 # Başlangıçtan 20 sn gecikmeden sonra ilk kontrol yapılır.
          periodSeconds: 3 # 3sn'de bir denemeye devam eder.
```
## Resource Management
- belirtilen request ile podun çalışması için gereken donanımı limit ile de podun çalısırken ki sınırlanırılamaıs ayarlanır.
```
resources:
      requests: # Podun çalışması için en az gereken gereksinim
        memory: "64M"	# Bu podu en az 64M 250m (yani çeyrek core)
        cpu: "250m" # = Çeyrek CPU core = "0.25"
      limits: # Podun çalışması için en fazla gereken limit
        memory: "256M"
        cpu: "0.5" # = "Yarım CPU Core" = "500m"
        
```
## Environment Vaiables
```
 env:
      - name: USER   # önce name'ini giriyoruz.
        value: "Ozgur"  # sonra value'sunu giriyoruz.
      - name: database
        value: "testdb.example.com"
 ```
 ## Volume
 - Containerın çalışma mantıgı içerisine girip bir şeyleri değiştrime gerekiyorsa sil baştan yarat. Bu şu şekilde daha iyi açıklanabilir: Container imajı oluşturulurken bir Readable only layer katmanı ve onun üzerinde de yazılabilir bir katman bırakır. Readevle only katmanı imaj kapsamındayken yazılabilir katman container kapsamındadır yani ben imajdan bir container oluşturdğumda her seferinde aynı R/O layer oluşusacak üzerinde yazılabilir bir katman eklşenecek.Ben container içerisine girip yazılabilir katmanda istediğim gibi dosya yüklerim veri yazarım fakat bu container öldüğünde artık o verilere ulaşamam.K8sde ki podlar da bir container olduguna göre aynı mnatık buradada gecerlidir. Bu durumu önlemenin 2 yolu var: 
 ### Geçici Volume
 - empty dir volumeler podlara özeldir. Podlar var oldugu sürece varlıklarını sürdürürler. Container ölse bile bu voluemler silinmez.Kulannım senaryosu olarak multicontainer olarak çalışan podlar örnek verilebeilir. Belirtilen pathlerdeki değişiklikler diğer container ppathinde de değişikliği neden olur.  
 ```
  volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      {}
 ```
 - hostPath volumeler podun üzerinde çalıştığı worker node un belirlenen pathlerini containerlara mount etmeye yarar. Worker node üzerinde değişiklikler kalıcıdır.Pod ölse bile..   
 ```
     volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /data
      type: Directory
 ```  
 
 ### Secret
 - Secret object uygulşamamızın için değerli ve gizli olan verileri güvenli(base64) bir şekilde tutacagımıuz object türüdür. Oluşturulan secret objesi de aynı volume gibi podlara baglanabilir,env variable olarak verilebilir.Secret obje örneği:  
 ```
 apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:		# Hassas data'ları stringData altına yazıyoruz.
  db_server: db.example.com
  db_username: admin
  db_password: P@ssw0rd!
```  
poda bağlama:  
```
   volumeMounts: # 2) Oluşturulan secret'lı volume'ü pod'a dahil ediyoruz.
    - name: secret-vol
      mountPath: /secret # 3) Uygulama içerisinde /secret klasörü volume'e dahil olmuştur. 
      # Artık uygulama içerisinden bu dosyaya ulaşıp, değerleri okuyabiliriz.
  volumes:				# 1) Önce volume'ü oluşturuyoruz ve secret'ı volume'e dahil ediyoruz.
  - name: secret-vol
    secret:
      secretName: mysecret3 
```
```
 env:	# Tüm secretları pod içerisinde env. variable olarak tanımlayabiliriz.
    # Bu yöntemde, tüm secretları ve değerlerini tek tek tanımladık.
      - name: username
        valueFrom:
          secretKeyRef:
            name: mysecret3 # mysecret3 isimli secret'ın "db_username" key'li değerini al.
            key: db_username
      - name: password
        valueFrom:
          secretKeyRef:
            name: mysecret3
            key: db_password
      - name: server
        valueFrom:
          secretKeyRef:
            name: mysecret3
            key: db_server
```
```
  envFrom: # 2. yöntemle aynı, sadece tek fark tüm secretları tek bir seferde tanımlıyoruz.
    - secretRef:
        name: mysecret3
```
### Config Map
 - Secret ile çalışma mantığı aynıdır. Tek fark secret base 64 crypted ile verileri tutar. 
### PV and PVC
 - Persistent volume : ephemeral volume olan emtydir de podlar öldüğünde volume yok oluyordur diğer volume olan hostPath de ise sadece üzerinde kostugu node üzerinde veri devamlılıgı saglanıyordu. Bu gibi durumlardan sıyrılmak adına nodelardan bağımsız bir volume oluşturabiliriz.  
 Not: CSI:konteyner orkestrasyon sistemlerinde depolama sağlayıcıları arasında ortak bir arayüz sağlar ve farklı depolama kaynaklarının daha kolay yönetilmesine olanak tanır.Default da gelen depolama sağlayıcıları dışında farklı depolama sağlayıcıları driverlarını sisteme yüklemek için bir arayüz sunar.  
 Persistent volume kullanabilmek için depolama sağlayıcısı ayaga kalkması gerekyor.Örnek bir nfs servisi:  
 ```
 apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
  labels:
    pv: deneme
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /tmp
    server: 172.17.0.2
 ```
 - Persistent volume claim : Persistent volumleri bir poda baglamak içic pvc lere ihtiyac vardır.Binevi aracı   
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  selector:
    matchLabels:
      pv: deneme
```
poda bağalama:  
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
### Storage Class
- Persistent cloume ile depolama sağlayıcıları kurma zahmatinden kurtulmak için tanımlanmış obje diyebiliriz.Clous servis saglayılarında var olan bu volumeler claim ile birlikte podlara mount edilebilir.  
PV PVC ve Strorage class için özet geçersek,stroage class aslında bizim fiziksel depolama birimimiz.Strogae Class üzerinden depolama sağlamak için persistent volumeler bir interface görevi görüyor.Pvc ise pv nin sağladığı bu arabirimden faydalanıp podlar ile baglantı kuruyor.Pv ile pvc nin eşleşmesi için StorageClassName ve AccesMode larının otomatik eşit olması gerekli.  
bizim geçici depolama olarak kullandığımız hostpathler ile sağladığımız depolama node ayakta kaldığı sürece var oluyor. Peki storage class ile localdisk  tipinde depolama sağlarkende hotspath ile node da bir dosyaya mount ediyoruz. Bu node ölünce veriler kaybolurmu yoksa disk hdd gibimi davranır.Depolama hdd gibi davranır.  
![image](https://user-images.githubusercontent.com/73287349/227888916-624fb2c3-c9d9-4bcb-b113-0bd4b9ee27bc.png)
![image](https://user-images.githubusercontent.com/73287349/227889518-97f609fd-3589-41cf-9fec-15936c163551.png)




## Authentication
- Dışarıdan bi kullanıcı cluster a bağlanmak için certfica(x-509) ile contexte bilgileri geçilerek authenticate olması gerekiyor.Önce istekte bulunacak kişi ssl key oluşturmalı ve onu crypte etmeli.Daha sonra bunu kuberntese yollamalı.Kuberntese geelen bilgieler dogrultusunda k8s admini kullanıcı için bir certifica oluşturur ve onu decode ederek kullanıcıya geri verir. Authenticate olan kullanıcın yetkileri 0 olarak gerlir.61.ders  
- RBAC = Role bir namespace içerisindeki objeler için(namespace belirtilmeli) , ClusterRole namespaceden bağımsız tüm cluster duzeyındeki objeler için rol atamaları yapar.
- Role:  
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-and-pod-logs-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list"]
```
- ClusterRole :  
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: aggregate-cron-tabs-view
  labels:
    # Add these permissions to the "view" default role.
    rbac.authorization.k8s.io/aggregate-to-view: "true"
rules:
- apiGroups: ["stable.example.com"]
  resources: ["crontabs"]
  verbs: ["get", "list", "watch"]
```  
- Role ve Cluster Rollerini kullanıcılara veya grouplara baglamak gerekir. Bunları yapacak objeler Rolebinding ve ClusterRoleBinding dir.  
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role 
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```  
### ServiceAccount
- Service account uygulamna bazında(pod yetkiside denebilir) api servere erişim için kullanılan bir hesap türüdür.Service accounta role ve roleBindingler ile role ataması yapılır. Container içerisinde oto oluşan certifica ve token sayesinde api-server e erişim sağlayabiliriz. Pipeline lar ile istenilern verileri elde edebiliriz.
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: testsa
  namespace: default
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: podread
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: testsarolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: testsa
  apiGroup: ""
roleRef:
  kind: Role
  name: podread
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: Pod
metadata:
  name: testpod
  namespace: default
spec:
  serviceAccountName: testsa
  containers:
  - name: testcontainer
    image: ozgurozturknet/k8s:latest
    ports:
    - containerPort: 80
 ```
 - Üstteki yamlde bir servisacoount ve rol oluşturulur. Rolebinding ile sa ya role ataması yapılıur.Pod içerisinde containerin servisaccount belirlenir.Bu sürecte pod oluşusrken /var/run/secrets/kubernetes.io/serviceaccount klasörü içeriisinde token certifica verileri oluşur. Bunları kullnarak api-server ile role seviyesinde etkileşim kurulabilir.
## DeamonSet
- Tüm workernode üzerinde bir pod çalışıtrmak istiyorsak bu podu deamonset ile sarmamız gerekir.Yeni bir node oluştugunda dahi deamonset bu podu orada tekrear ayaga kaldırır.Yaml formatı deployment ile tamamen aynıdır.
## k8s Ağ altyapısı
K8s kurulumda pod’lara ip dağıtılması için bir IP adres aralığı (--pod-network-cidr) belirlenir.Tüm podlara uniq bir ip atanır.
Aynı cluster üzerindeki poldar birbirleri üzerinde bir Nat olmadan haberleşebilir.3 tür servis objesi vardır:
### Services 
- ClusterIP: Cluster içöerisindeki podlar için bir endpoint oluşturur. Tüm podlar birbirlerinin adlarını tek tek bulmak yerine bu servis objesinin sağladığı endpointlerdenb haberleşebilir.  
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
```  
-NodePort: kümr dışındaki harici bir ip adresiyle ileitişim kurulmak istenebilir.Bu pod türü sayesinde sağlanır.  
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - port: 80
      targetPort: 80
```  
- LoadBalancer: Bir bulut sağlayıcının yük dengeleyicisini kullanarak Hizmeti harici olarak kullanıma sunar.
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  ```  
  ![image](https://user-images.githubusercontent.com/73287349/227898515-2030bc47-543b-4ca9-8390-e34e5f79c6fc.png)  

  impretaive şekilde servis expose etme:
  ```kubectl expose deployment backend --type=ClusterIP --name=backend```
  -DNS = IPAdress.namespace.objecttype.cluster.local
###  NetworkPolicy
  Bu obje türü slector ile podları seçerek onların ağ trafikteki rollerini belirtir. Örneğin dışarıdan gelen(ingres) hangi poda yönlendirileceği, dışarıya giden(egres) trafiğin hangi podlar üzerinde olacagını beliritr.  
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```  
### Ingrees
- Uygulamaları dış dunyaya açarken 1. sorun her uygulama içinb farklı farklı IP almak maaliyet,ikinci sorun mikroservice bir mimariye göre çalışan uygulamamda http://localhost/a a gelen ve http://localhost/b ye gelen istekler doğrultusunda trafiği container içerisinde farklı pathlere dağıtmak mümükün değil. Bu iki sorunudaortadan kaldıran ingreestir. 
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "*.foo.com"
    http:
      paths:
      - pathType: Prefix
        path: "/foo"
        backend:
          service:
            name: service2
            port:
              number: 80
```  
![image](https://user-images.githubusercontent.com/73287349/227898397-65b21924-98be-4a5c-947b-137ac6de3ed7.png)


# Cloud Guru

## NameSpace 
- kubectl get namespaces
- kubectl get ns
- kubectl get pod --namespace dev
- kubectl get pod -n dev
- kubectl get pod --all-namespace
- kubectl get pod -A
- kubectl create ns dev
- kubectl delete ns dev
- kubectl run pod firstpod --image=nginx --namesapce=dev
- YAML: 
- - ns.yml =
```
    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
```
- - kubectl create -f ns.yml
- - kubectl apply -f ns.yml

## safely node drain
- Bir node un bakım aşaması geldiğinde node ugüvenli bir şekilde clusterdan kopratılması gerekir. İlk önce node u skubecsheduler tarafıdan schedule edilemez hale getirebilriz ya da direkt drain edebiliriz. 
- kubectl cordon node1 = Node1 artık kubescheculer tarafından schedule dielemez. Yabni herhangi bir pod bu node a atanamaz.
- kubectl drain  node1 =  "ReplicationController, ReplicaSet, Job veya StatefulSet tarafından yönetilmeyen Bölmeler silinemez". Yani direkt olarak manuel bir pod oluşturmada node u drain etmek gerekirken hata alırız. Bunu önlemek için kubectl drain  node1 --force kullanılmalı. Drain edilecek node üzerinde deamonset var ise yine hata alırız. Deamonsetleri gormezden gelmek için kubectl drain node1 --ignore-daemonsets kullanılmalı.
- kubectl uncordon node1 = Bakımı tamamlanmış node u clustera tekrar ekler fakat silinmiş yada evict edilmiş podlar tekrar o node uzerine geçmez.
- node selector ile bir node seçip pod yarat.
- aynı şekilde deployment yarat
- node u drain etmeye çalış: kuebctl drain nodename
- hata gelicek pod = "ReplicationController, ReplicaSet, Job veya StatefulSet tarafından yönetilmeyen Bölmeler silinemez"
- hem daemonsetden hemde "ReplicationController, ReplicaSet, Job veya StatefulSet tarafından yönetilmeyen Bölmeler silinemez" bunları görmezden gelmek için
- kubectl drain worker-2 --ignore-daemonsets --force
- worker-2        Ready,SchedulingDisabled   <none>          30m   v1.24.0
- deployment otomatik oalrak farklı nodelara dağılır.(ben nodeselector ile seçtiğim için dağılmadı)
- kubectl uncordon worker-2 ile node eski haline döner.
- kubedoc da aynı
## Upgrading kubeadm
- step by step upgrade işlemi:
- - kubectl drain <controlplaneName> --ignore-deamonsets
  - sudo apt-get update
  - sudo apt get install -y --allow-change-held-packages kubeadm=1.22.2-00
  - kubeadm version
  - sudo kubeadm upgrade plan v1.22.2 (v1.22.2 git version)
  - sudo kubeadm upgrade apply v1.22.2
  - sudo apt-get update
  - sudo apt get install -y --allow-change-held-packages kubelet=1.22.2-00 kubectl=1.22.2-00 
  - sudo systemctl daemon-reload
  - sudo systemctl restart kubelet
  - kubectl uncordon <controlplaneName>
  - kubectl drain <workernodename> --ignore-deamonsets --force
  - sudo apt-get update
  - sudo apt get install -y --allow-change-held-packages kubeadm=1.22.2-00
  - sudo kubeadm upgrade node
  - sudo apt-get update
  - sudo apt get install -y --allow-change-held-packages kubelet=1.22.2-00 kubectl=1.22.2-00 
  - sudo systemctl daemon-reload
  - sudo systemctl restart kubelet
  - kubectl uncordon <workenrnodename>
  - kube doc da var aynısı
  - ilk önce control plnae drain et
  - sonra apt kubeadm
  - upgradeplan and upgrade apply
  - sonra kubelet kubcetl apt
  - sudo systemctl daemon-reload sudo systemctl restart kubelet
  - sonra uncordon
  - worker node u kontrol de drain et
  - sonra kubeadm apt
  - upgrade node
  - sonra kubelet kubcetl apt
  - sudo systemctl daemon-reload sudo systemctl restart kubelet
  - sonra uncordon
## ETCD BackUp
  
  
## chp4
- ```
  kubectl apply -f ..yml (var olan pod a bir daha create dersen hata almaz değiştirir)
  kubectl create -f ..yml (var olan pod a bir daha create dersen hata)
  kubectl get pod -o (wide,yaml,json)
  kubectl get pod --selector app = dev (etiketlere göre seçer)
  kubectl get pod -n kube-system
  kubectl get pods -o yaml -n kube-system --sort-by .spec.dnsPolicy (sort by sıralama)
  kubectl exec my_pod -- echo "hi"
  kubectl exec multipod -c containername -- echo "hi"
  kubetl delete pod pod1
  kubectl run pod --image=nginx --dry-run -o yaml
  kubectl create deployment mydeployment..
  kubectl scale deployment mydeployment --replica-set=2 --record (--record kaydediro)
  ```
- Role: Namespace kapsamında kullanıcılara yetkiler vermek üzere oluşturulur.Role oluşturduktan sonra kullanıcıya bağlamak için Rolebinding objesi kullanılır.
- Cluster Role: Cluster kapsamında kullanıcılara yetkiler vermek üzere oluşturulur.Role oluşturduktan sonra kullanıcıya bağlamak için ClusterRolebinding objesi kullanılır.
- Service Account: Podlarında bazen k8s apisine ulaşması gerekebilir. Bunu yapmak için bir kaç authemtication işlemi yapması gerekiyor. Genel işleyiş Rollerin Role binding ile bir servis accounta baglanıp servis accountunda bir poda bağlanması. Bu şekilde yetkilendirilen podun /var/run/secrets/kube.. altında sertifikası tokenı vs oluşuyor. Artık pod bu token ile k8s apisini kullanabilir. KUBDOC DA YOK. kubectl create sa deneme --dry-run -o yaml
- kubectl top pod,kubectl top node =) Memory Cpu kullanımları gösterir.
- kubectl top pod,kubectl top node --sort-by cpu
## chapter5
- configMap = poda bilgi geçmek istersek.
- secret = poda bilgi geçmek istersek.(base 64)

## extralar
- #### Affinitiy
- Bu affinity ile seçilen node var ise podu oluştur. Yoksa oluşturma filtresi yapılıyor. Zorunlu tutarak.
- ![image](https://github.com/omerulusoy41/Kubernetes/assets/73287349/d648f166-2b71-4ba5-bc6f-44e6ae340459)
- bu affinity ile belirtilen node varmı yokmu kontrol edilir yoksa podu yine çalıştırır.()weight öncelik
- ![image](https://github.com/omerulusoy41/Kubernetes/assets/73287349/b61f0b23-8197-4920-a4d7-35c7fc5018b1)
- pod affinity ile filtrelenmiş özelliklere göre worker node üzerinde filtreyi uygulayarak podu calıstırır. Yani filtre o worker nodda var mı varsa çalışıtır. Topologykey: hostname,AZ vs. belirlemek için.
- ![image](https://github.com/omerulusoy41/Kubernetes/assets/73287349/4e2f025e-6531-4f81-9f8d-a0583915abe4)
- #### Statefulset
- headles Service:
- ```
   apiVersion: v1
    kind: Service
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      ports:
      - port: 80
        name: web
      clusterIP: None
      selector:
        app: nginx
  ```
  - bu şekilde arka tarafda podlara hizmet eden  headles service i farklı bir pod oluşturup pinglersek yine loadbalancing ile farklı farklı podları pingler fakat headless service in sağladıgı extra özellik ise podname.servicename yaparak belirli poda hizmet etmesidir.
  - statefulset: Bu obje türü deploymenta cok benzer fakat deploymenta extra olarak podlar için unique name ve sıralama yaparak bir ordering tanımlar. pv ler ile desteklenirse silinen pod yerine yine aynı özellikte aynı pod aynı değerlerle eşitlenir.Scale up Scale down da yine order bir şekilde devam eder.
 
  - #### Job
  -  Job: Anlık çalışıp kapanacak podlar için örneğin veritabanı pod um var ve ben bunun içinden veri alıp exelde maplicam bunun için singleton pod tanımlayıp deneyebilirim fakat singleton pod fail edebilir ederse düzeltiilemez bunun yerine deployment ile denesem işim bittiğinde podlar silinmez silinirse tekrar oluşur bunlar yerine job nesneleri oluşturarak pod işi tamamlanınca kapanan podlar tanımlayabiliriz.
  -  ![image](https://github.com/omerulusoy41/Kubernetes/assets/73287349/5e6ae38a-3edb-452d-bf96-a4b3de9ca16e)
  -  cronjob: Periyodik joblar için shcedule düzenler.
  -  ![image](https://github.com/omerulusoy41/Kubernetes/assets/73287349/fa118a0d-9527-447f-8e72-070ae8755fac)
  -  #### private Reristry
  -  private registry var ise kubeletin imajları registryden çekmesi için bir authentication işlemi yapması gerekiyor. Biz bunu pod tanımı içerisinde sercretlar verererk hallediyoruz. Öncesinde bir secret olusturuyoruz. Secretın tipi docker-registy şeklinde
  -  kubectl create secret docker-registry name --docker-server="url" --docker-username="kuLlanici_adi"--docker-password="sifre" olucak. poda tanımlanması da:
  -  ![image](https://github.com/omerulusoy41/Kubernetes/assets/73287349/701cc045-3b19-4d98-a9d4-5f92192e5ef9)
  - #### Helm
  - Helm,Kubernetes üzerinde uygulamaları kolayca yönetmenizi yarayan bir araç olarak karşımıza çıkıyor.Helm ile kolayca deploy edebilir,upgrade edebilir,sürümleri kontrol edebilirsiniz.Helm ürünü ile birlikte “Charts” dediğimiz kavram giriyor.Charts,önceden Kubernetes uygulamalarımızı deploy etmek için hazırladığımız yaml dosyaları bütünüdür.Service,Deployment,ReplicationController gibi tanımlamaların paket hali diyebiliriz.








 











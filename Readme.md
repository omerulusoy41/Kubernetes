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
- kind –> Hangi object türünü oluşturmak istiyorsak buraya yazarız. ÖR: pod  
- apiversion -> Oluşturmak istediğimiz object’in hangi API üzerinde ya da endpoint üzerinde sunulduğunu gösterir.  
- metadata -> Object ile ilgili unique bilgileri tanımladığımız yerdir. ÖR: namespace, annotation vb.  
- spec -> Oluşturmak istediğimiz object’in özelliklerini belirttiğimiz yerdir. Her object için gireceğimiz bilgiler farklıdır. Burada yazacağımız tanımları, dokümantasyondan bakabiliriz.  
- oluşturulan yaml dosyasını çaliştırma : ```kubectl apply -f <...yml>```  
- oluşturulan yaml dosyasını silme : ```kubectl delete -f <...yml>```  
- oluşturulan yaml dosyasını editleme : ```kubectl edit -f <...yml>```  
### pod yaşam döngüsü
- Bir pod oluşturulacakken önce etdye kaydedilir(Pending).Daha sonra podun üzerinde koşacağı node kube-schduler tarafından ayarlanır ve etcd ye kaydedilir.Daha sonra
etcd kayıtlı olan bilgileri izleyen kubelet containerd ile imajları oluşsturuur.  
Pod klasörünü incele...









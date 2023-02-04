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

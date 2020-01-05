## Prasyarat
Setelah selesai mensetup DigitalOcean kubernetes cluster.

## Catatan
Untuk melihat dokumentasi wordpress docker cek disini: https://hub.docker.com/_/wordpress/

Hal yang perlu diperhatikan:
- environment variables yang digunakan container (INPUT)
- proses apa saja yang berjalan di container.
- service ports yang diekspos oleh container (OUTPUT)


### Diagram Infrastructur:
Pada sisi kiri adalah tradisional diagram untuk aplikasi web 3-tier. Pada sebelah kanan, anda akan melihat bagaimana setiap bagian infrastuktur di petakan ke konsep kubernetes.

    Config data            	(k8s ConfigMaps and Secrets)

    MySQL Container             (k8s replicaset)
    MySQL Service               (k8s service)
        |
        |
    WordPress Container         (k8s deployment)
    [ apache dan php-fpm ]
        |
        |
    DO Loadbalancer             (k8s service)


## 1. Mysql Setup

File Secret membutuhkan nilai base64-encoded values jika ingin digunakan dengan cara yang sama

Generate MYSQL_PASSWORD baru and nilai MYSQL_ROOT_PASSWORD seperti ini dan gantikan nilai nya disini secrets/wp-mysql-secrets.env:

    echo && cat /dev/urandom | env LC_CTYPE=C tr -dc [:alnum:] | head -c 15 | base64 && echo


Now, Buat sebuah all-in-one secret:

    kubectl apply -f secrets/wp-mysql-secrets.yaml


Buat mysql volume dan replicaset anda. Ekspos ini ke internal service yang baru.

    kubectl apply -f manifestasi/mysql-volume-claim.yaml
    kubectl apply -f manifestasi/mysql-replicaset.yaml
    kubectl apply -f manifestasi/mysql-service.yaml


Jalankan shell didalam mysql container, login kedalam mysql, dan setup Database:

    kubectl get pods			    # Dapatkan nama pods
    kubectl exec -it mysql-abcde -- bash    # Ganti mysql-abcde dengan nama pod yang asli.

    # Gunakan password root yang dibuat diawal(secret/wp-mysql-secrets.yaml)
    mysql -u root -p

    #Pada shell mysql anda:
    CREATE DATABASE wordpress;

Ctrl-d untuk kembali.


Periksa apa yang telah kita buat!

    kubectl get pv
    kubectl get secrets
    kubectl get replicasets
    kubectl get pods
    kubectl describe pod $PODANDA
    kubectl logs $PODANDA


## 2. Wordpress Setup

Edit config file di configs/apache.conf jika kita ingin menggunakan nama domain untuk situs wordpress kita.

    # Jika kita menggunakan custom apache config pada container kita
    # kita bisa menggunakan:
    # kubectl create cm --from-file kondigurasi/apache.conf apache-config

    kubectl apply -f manifestasi/wordpress-datavolume-claim.yaml
    kubectl apply -f manifestasi/wordpress-deployment.yaml

Periksa pola untuk mendapatkan single file config kedalam container di wordpress-deployment.yaml.


## 3. Load Balancer Setup
DigitalOcean Load Balancer berperan untuk mengekspos cluster kita ke internet.

    kubectl apply -f manifestasi/wordpress-loadbalancer.yaml


## Lihat Hasil deployment!
Dapat kan load balancer external IP dengan perintah berikut:

    kubectl get services

Kita juga bisa melihat melalui DigitalOcean Dashboard: Networking --> Load Balancers.


## Cleanup
Untuk menghapus semua dan mengulang proses pembuatan, pergi ke dashboard DigitalOcean anda:

1. Hapus kubernetes cluster
1. Networking --> Load Balancers --> LB Settings --> Delete
1. Volumes --> Delete your volumes

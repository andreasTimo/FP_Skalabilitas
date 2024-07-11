
# FINAL PROJECT SKALABILITAS

## Anggota
**Andreas Timotius P. S**  
5027211019
** Muhammad Hilmi Azis**
5027201049
**Maulanal Fatihil A.M.**
5027201031
**Made Krisnanda Utama**
05311840000033

## Tampilan Website

![image](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/1f427938-c767-4520-aceb-96a70cd6941d)


### Arsitektur
#### Arsitektur Nodes
Menggunakan 4 Nodes dibagi menjadi 1 Master dan 3 Worker

![FPSkala-Arsitektur Node drawio](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/157acf35-3fe1-4a3c-ba3e-a4db9da264d5)


```
Master: 10.15.40.67 : skalabilitas
Worker:
-10.15.40.55 : worker-node-3
-10.15.40.59 : worker-node-4
-10.15.40.64 : worker-node-2
```

#### Arsitektur Aplikasi
Service Aplikasi digunakan untuk menghubungkan ke 5 pods masing-masing tiap apps

![FPSkala-Page-2 drawio](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/9594020d-c44a-4133-8a40-f3aae5ad28ef)


### Langkah-Langkah Deploy Aplikasi
1. **Menginstall microk8s**
   Pada masing-masing nodes, install microk8s dengan command ini:
   ```bash
   sudo snap install microk8s --classic
   sudo usermod -aG microk8s $USER
   sudo chown -f -R $USER ~/.kube
   newgrp microk8s
   microk8s status --wait-ready
   ```
   Pastikan `microk8s status --wait-ready` sudah selesai.

2. **Tambahkan Nodes**
   Pada master, tulisakan `microk8s add-node` yang akan mengeluarkan output seperti ini dan masukan ke masing-masing worker.

   Tambahkan juga label worker ke masing-masing nodes dengan:
   ```bash
   kubectl label nodes worker-node-2 node-role.kubernetes.io/worker=worker
   kubectl label nodes worker-node-3 node-role.kubernetes.io/worker=worker
   kubectl label nodes worker-node-4 node-role.kubernetes.io/worker=worker
   ```
   Cek dengan `kubectl get nodes`, kalau berhasil output akan seperti ini:

3. **Enable Prometheus**
   Enable Prometheus dengan cara `microk8s enable prometheus`. Kalau sudah berhasil cek dengan `kubectl get all -n observability` yang akan mengeluarkan hasil.

   ![image](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/62f741f0-0df8-4802-a728-3bc8cfdf49e3)


5. **Apply semua service**
   Untuk mengetahui API, kita harus mengapply service dulu sebelum deployment.

   Untuk [frontend](https://github.com/andreasTimo/FP_Skalabilitas/blob/main/source/sa-frontend-service.yaml), [Webapp](https://github.com/andreasTimo/FP_Skalabilitas/blob/main/source/sa-webapp-service.yaml), [Logic](https://github.com/andreasTimo/FP_Skalabilitas/blob/main/source/sa-logic-service.yaml)

   Cek dengan `kubectl get svc`, jika berhasil output akan seperti ini:
   ![image](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/07426b32-441d-423b-a9d5-e11bb4cd83a5)


7. **Masukan API ke tiap-tiap aplikasi**
   Ketika sudah dapat, bisa mengubah tiap-tiap aplikasi.

   Untuk frontend, ubah pada `/sa-frontend/src/App.js`:
   ```javascript
   fetch("http://<ipnodemaster>:<portservicewebapp>/sentiment", {
     method: "POST",
     headers: {
       "Content-Type": "application/json"
     },
     body: JSON.stringify({ text: inputText })
   })
   ```

   Untuk webapp, ubah pada `/sa-webapp/src/main/java/com/sa/web/SentimentController.java`:
   ```java
   String url = "http://<ipnodemaster>:<portservicelogic>/analyse/sentiment";
   ```

   Untuk logic, tidak ada yang perlu dirubah. Jangan lupa untuk dibuild dan dipush ke docker agar image dipakai pada deployment.

8. **Deploy tiap aplikasi**
   Buat `sa-frontend-deployment.yaml` dan isi seperti dibawah ini:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: sa-frontend
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: sa-frontend
     template:
       metadata:
         labels:
           app: sa-frontend
       spec:
         containers:
         - name: sa-frontend
           image: andreast3/sentiment-analysis-frontend
           ports:
           - containerPort: 80
   ```
[Frontend-Deployment](https://github.com/andreasTimo/FP_Skalabilitas/blob/main/source/sa-frontend-deplyoment.yaml)

   Buat `sa-webapp-deployment.yaml` dan isi seperti dibawah ini:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: sa-webapp
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: sa-webapp
     template:
       metadata:
         labels:
           app: sa-webapp
       spec:
         containers:
         - name: sa-webapp
           image: andreast3/sentiment-analysis-webapp
           ports:
           - containerPort: 8080
   ```
[WebApp-Deployment](https://github.com/andreasTimo/FP_Skalabilitas/blob/main/source/sa-webapp-deployment.yaml)

   Dan terakhir buat `sa-logic-deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: sa-logic
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: sa-logic
     template:
       metadata:
         labels:
           app: sa-logic
       spec:
         containers:
         - name: sa-logic
           image: andreast3/sentiment-analysis-logic
           ports:
           - containerPort: 5000
   ```
[Logic-Deployment](https://github.com/andreasTimo/FP_Skalabilitas/blob/main/source/sa-logic-deployment.yaml)

   Apply dengan `kubectl apply -f <file yaml>`.

   Kalau berhasil cek dengan `kubectl get pods` dan `kubectl get deployment`.

   
   ![image](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/ee87e058-8a4c-4c16-9dcf-033dd027046c)



9. **Monitoring**
   Untuk membuka Grafana dan Prometheus, kita harus mengubah `kube-prom-stack-grafana` dan `kube-prom-stack-kube-prome-prometheus` menjadi NodePort dengan command dibawah ini:
   ```bash
   kubectl patch svc kube-prom-stack-grafana -n observability --patch '{"spec": {"type": "NodePort"}}'
   kubectl patch svc kube-prom-stack-kube-prome-prometheus -n observability --patch '{"spec": {"type": "NodePort"}}'
   ```

   Lalu jika berhasil dicek dengan `kubectl get svc kube-prom-stack-kube-prome-prometheus -n observability`.

   
![image](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/c2c33ffa-1095-4076-9288-7e80876209bb)


   Lalu buka web dengan `<ipnodemaster>:<port yang tertera>`.

11. **Grafana dan Prometheus**
   Ketika masuk Grafana masukan username: `admin` dan password: `prom-operator`, lalu buka bagian `dashboard → browser`.

![image](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/7225cc53-c9ac-448b-b143-9d0a86543727)




   Di sini secara otomatis terdeteksi. Jika ingin melihat resource tiap pods yang dipakai dalam tiap node ada di:

![image](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/44dff5eb-3f1f-43eb-ad83-c6e06f46bd2c)

![image](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/6b6b7e00-69ed-4722-a268-adaebb560bf8)


## Kesimpulan
Berdasarkan proyek akhir ini, mendeploy aplikasi pada cluster Kubernetes multi-node terbukti menantang. Menghadapi beberapa kesulitan, seperti kebingungan dalam merancang arsitektur yang sesuai dan menentukan endpoint API yang efektif, seringkali mengharuskan saya untuk membuild ulang aplikasi. Selain itu, terdapat kendala dalam img pulling yang disebabkan oleh masalah koneksi internet. Karena itu, total traffic yang dapat ditangani sistem ini tanpa kegagalan belum dapat ditentukan. Meskipun membutuhkan load testing lebih lanjut, keterbatasan waktu dan kesulitan konfigurasi membuatnya sulit dilakukan saat ini. 




   

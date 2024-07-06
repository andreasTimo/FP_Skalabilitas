# FINAL PROJECT SKALABILITAS

## Anggota
**Andreas Timotius P. S**  
5027211019

## Tampilan Website
![image](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/1f427938-c767-4520-aceb-96a70cd6941d)

### Arsitektur
#### Arsitektur Nodes
Menggunakan 4 Nodes dibagi menjadi 1 Master dan 3 Worker

![FPSkala-Arsitektur Node drawio](https://github.com/andreasTimo/FP_Skalabilitas/assets/56831859/157acf35-3fe1-4a3c-ba3e-a4db9da264d5)


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

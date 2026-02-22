```
# 1. Запускаем кластер (выделяем 10GB RAM и 4 CPU)
minikube start --cpus 4 --memory 10240 --driver=docker --disk-size=50g

# 2. Включаем Ingress (чтобы открывать сайты через домен, а не IP:Port)
minikube addons enable ingress

# 3. В ОТДЕЛЬНОМ окне PowerShell (от админа) запускаем туннель.
# ЭТО ОКНО НЕ ЗАКРЫВАТЬ! Оно пробрасывает сеть в Windows.
minikube tunnel
```
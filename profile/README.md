## Instrukcja na instalcje kubernetesa

zakladajac ze jest sie w folderze ../efarm i wszystkie repa sa w: 
- ../efarm/efarm-backend 
- ../efarm/efarm-frontend
- ../efarm/efarm-database

### Wymagania:
- Docker
- Kubectl
- Helm
- Dystrybucja kubernetes(k3d tutaj)

### Instalacja Docker:
tutaj pomijam instrukcje instalacji bo zakladam ze jest juz obecny

### Instalacja Kubectl:
- MacOS:
https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/
- Linux:
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- Windows:
https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

### Instalacja Helm:
Ja pobieralem ze skryptu ale widze ze jest opcja pobrania z brew nizej:
https://helm.sh/docs/intro/install/

### Instalacja k3d:
https://k3d.io/v5.7.3/#install-script

### Rozpoczecie pracy z k3d:
Zeby utworzyc cluster:

```sh
k3d cluster create my-cluster --servers 1 --port 8080:80@loadbalancer
```

Teraz aplikacja bedzie dostepna na porcie 8080 oczywiscie jezeli zajety mozna zmienic w komendzie na inny i dalej bedzie dzialac tylko na innym porcie.

```sh
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets -n kube-system --set-string fullnameOverride=sealed-secrets-controller sealed-secrets/sealed-secrets
```

Uzywajac pliku sealed-secret-key.yaml ktory nie jest w zadnym repo ale musi byc dodany zrobic samemu:

```sh
kubectl apply -f sealed-secrets-key.yaml
```

### Zarzadzanie aplikacja przy uzyciu Helm
Zeby odpalic aplikacje trzeba uzyc podanych komend:

```sh
helm install efarm-mysql ./efarm-mysql/helm 
```

```sh
helm install efarm-backend ./efarm-backend/helm 
```

```sh
helm install efarm-frontend ./efarm-frontend/helm 
```

### Aktualizacja aplikacji przy uzyciu Helm
Zeby usunac aplikacje gdy nie jest potrzebna np. dla backendu:

```sh
helm uninstall efarm-backend
```

Jezeli aplikacja ciagle byla zainstalowana i dzialala to zeby ja zaktualizaowac zeby uzywala najnowszego obrazu docker trzeba np. dla backendu:

```sh
helm upgrade efarm-backend ./efarm-backend/helm
```


### Sprawdzenie statusu aplikacji
Aby upewnic sie ze baza danych dziala mozna uzyc podanej komendy aby poczekac przez 120s na gotowosc zasobow:

```sh
kubectl wait --for=condition=ready pod -l app=efarm-mysql --timeout=120s
```

Jezeli pojawi sie:
- pod/<app-name> condition met

oznacza ze juz dziala
- error: timed out waiting for the condition on pods/<app-name>

to znaczy ze jeszcze nie jest gotowe wiec albo powtorzyc komende albo sprawdzic stan z komenda:

```sh
kubectl get pods
```

Zeby wejsc do konteneru tak jak to mozna z docker exec trzeba:

```sh
kubectl exec -it <pod-name> -- /bin/bash
```

### Zarzadzanie klastrem

- **Zatrzymanie klastra**:

```sh
k3d cluster stop my-cluster
```

- **Ponowne urochomenie klastra**:

```sh
k3d cluster start my-cluster
```

- **Usuniecie klastra** (w momenie ktorym nie jest potrzebny):

```sh
k3d cluster delete my-cluster
```


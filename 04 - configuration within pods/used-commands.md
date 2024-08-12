> kubectl apply -f sleep/sleep.yaml

> kubectl exec deploy/sleep -- printenv HOSTNAME KIAMOL_CHAPTER 

> kubectl apply -f sleep/sleep-with-env.yaml

Komenda apply służy do aktualizacji wdrożeń, nie są tworzone nowe a aktualizowane te, które zostaną dopasowane po selektorze

>kubectl exec deploy/sleep -- printenv HOSTNAME KIAMOL_CHAPTER 

> kb create configmap sleep-config-literal --from-literal=kiamol.section='4.1'  
configmap/sleep-config-literal created

> kb get configmap
NAME                   DATA   AGE
kube-root-ca.crt       1      35h
sleep-config-literal   1      8s

> kb describe cm sleep-config-literal
Name:         sleep-config-literal
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
kiamol.section:
----
4.1

BinaryData
====


> kb apply -f .\sleep\sleep-with-configMap-env.yaml  

> kubectl exec deploy/sleep -- sh -c 'printenv | grep "^KIAMOL"' 
KIAMOL_SECTION=4.1
KIAMOL_CHAPTER=04

> kb create cm sleep-config-env-file --from-env-file=sleep/ch04.env

> kb apply -f .\sleep\sleep-with-configMap-env-file.yaml
deployment.apps/sleep configured

> kubectl exec deploy/sleep -- sh -c 'printenv | grep "^KIAMOL"' 
KIAMOL_EXERCISE=try it now
KIAMOL_SECTION=4.1
KIAMOL_CHAPTER=04

Jeżeli istnieją zduplikowane klucze, zmienne środowiskowe zdefiniowane za pomocą `env` nadpisują wartości zdefiniowane za pomocą `envFrom`

-----

## Aplikacja Todo 

> kubectl apply -f .\todo-list\todo-web.yaml

> kb get svc todo-web

> kb logs -l app=todo-web

> kb apply -f .\todo-list\configMaps\todo-web-config-dev.yaml

> kubectl apply -f .\todo-list\todo-web-dev.yaml

# Wyświetl domyślny plik konfiguracyjny: 
> kubectl exec deploy/todo-web -- sh -c 'ls -l /app/app*.json' 

# Wyświetl plik konfiguracyjny z punktu montowania woluminu: 
> kubectl exec deploy/todo-web -- sh -c 'ls -l /app/config/*.json' 

# Sprawdź, czy plik jest rzeczywiście tylko do odczytu: 
> kubectl exec deploy/todo-web -- sh -c 'echo ch04 >> /app/config/config.json' 

- aktualizacja configmapy podczas zycia poda
kb apply -f .\todo-list\configMaps\todo-web-config-dev-with-logging.yaml

- obserwowanie wynikow, gdzie plik zostal dodany
kubectl exec deploy/todo-web -- sh -c 'ls -l /app/config/*.json'

- obserwowanie wynikow, gdzie zmiany w pliku zostały uwzględnione w aplikacji (/config)
kb logs -l app=todo-web


----
Próba uruchomienia aplikacji z nieprawidłowym zamontowaniem configmapy, skutkujące nadpisaniem katalogu z plikami aplikacji

> kb apply -f .\todo-list\todo-web-dev-broken.yaml
deployment.apps/todo-web configured

> kb logs -l app=todo-web
  * You intended to execute a .NET application:
      The application 'ToDoList.dll' does not exist.
  * You intended to execute a .NET SDK command:
      No .NET SDKs were found.

> kb get pods -l app=todo-web
NAME                        READY   STATUS             RESTARTS      AGE
todo-web-559c9fb5f7-fg9v7   1/1     Running            0             14m
todo-web-fd8cbdd6-pdkmn     0/1     CrashLoopBackOff   2 (31s ago)   53s



> kb apply -f .\todo-list\todo-web-dev-no-logging.yaml
deployment.apps/todo-web configured

> kb get pods -l app=todo-web
NAME                        READY   STATUS    RESTARTS   AGE
todo-web-85c6bc54b5-7ccsd   1/1     Running   0          7s



----
## Sekrety

> kb create secret generic sleep-secret-literal --from-literal=secret=shh...
secret/sleep-secret-literal created

> kb describe secret sleep-secret-literal
Name:         sleep-secret-literal
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
secret:  6 bytes
> kb get  secret sleep-secret-literal    
NAME                   TYPE     DATA   AGE
sleep-secret-literal   Opaque   1      24s

> kb get  secret sleep-secret-literal -o jsonpath='{.data.secret}'
c2hoLi4u // zakodowana wartość sekretu, kubernetes przechowuje sekrety zakodowane w base64

> kb get  secret sleep-secret-literal -o jsonpath='{.data.secret}' | base64 -d

> function Convert-ToBase64 {
>>     param (
>>         [parameter(ValueFromPipeline)]
>>         [string] $text,
>>         [switch] $d
>>     )
>>     if ($d){
>>         [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($text))
>>     }
>>     else {
>>         [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($text))
>>     }
>> }
> Set-Alias base64  Convert-ToBase64

> kb get  secret sleep-secret-literal -o jsonpath='{.data.secret}' | base64 -d
shh...
> 

--- 
Uruchomienie poda k8s korzystającego z sekretu środowiskowego zdefiniowanego powyżej, przypisującego go jako zmienna środowiskowa
> kb apply -f .\sleep\sleep-with-secret.yaml
deployment.apps/sleep configured

> kb exec deploy/sleep -- printenv KIAMOL_SECRET
shh...

> kb apply -f .\todo-list\secrets\todo-db-secret-test.yaml
secret/todo-db-secret-test configured

> kb get secret todo-db-secret-test -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 -d
kiamol-2*2*
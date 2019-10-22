oc explain dc --recursive=true // This outputs the full yaml file description
oc set resource -h // shows the help function for resources

oc new-app -e POSTGRESQL_USER=user37,POSTGRESQL_PASSWORD=user37,POSTGRESQL_DATABASE=user37, VOLUME_CAPACITY=5Gi registry.redhat.io/rhscl/postgresql-95-rhel7

oc new-app postgresql-persistent --param POSTGRESQL_DATABASE=gogs --param POSTGRESQL_USER=gogs --param POSTGRESQL_PASSWORD=gogs --param VOLUME_CAPACITY=4Gi -lapp=postgresql_gogs

oc new-app wkulhanek/gogs:11.34

//Set deployment strategy to Recreate
oc patch dc gogs --patch='{ "spec": { "strategy": { "type": "Recreate" }}}'

oc new-app -e SONARQUBE_JDBC_USERNAME=user37 SONARQUBE_JDBC_PASSWORD=user37 POSTGRESQL_DATABASE=user37 SONARQUBE_JDBC_URL=jdbc:postgresql://postgresql/user37 wkulhanek/sonarqube:6.7.6

oc set volume dc/gogs --add --overwrite --name=gogs --mount-path=/data/ --type persistentVolumeClaim --claim-name=gogs-pvc --claim-size=5Gi

oc set volume dc/sonarqube --add --overwrite --name=sonarqube-volume-1 --mount-path=/opt/sonarqube/data/ --type persistentVolumeClaim --claim-name=sonarqube-pvc --claim-size=5Gi

oc set resources dc/sonarqube --limits=memory=3Gi,cpu=2 --requests=memory=2Gi,cpu=1

oc patch dc sonarqube --patch='{ "spec": { "strategy": { "type": "Recreate" }}}'

oc set probe dc/sonarqube --liveness --failure-threshold 3 --initial-delay-seconds 40 -- echo ok
oc set probe dc/sonarqube --readiness --failure-threshold 3 --initial-delay-seconds 20 --get-url=http://:9000/about


apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "limits" 
spec:
  limits:
    - type: "Pod"
      max:
        cpu: "2" 
        memory: "1Gi" 
      min:
        cpu: "200m" 
        memory: "6Mi" 
    - type: "Container"
      max:
        cpu: "2" 
        memory: "1Gi" 
      min:
        cpu: "100m" 
        memory: "4Mi" 
      default:
        cpu: "300m" 
        memory: "200Mi" 
      defaultRequest:
        cpu: "200m" 
        memory: "100Mi" 
      maxLimitRequestRatio:
        cpu: "10" 
# Deploy Prometheus and Grafana on Kubernetes
Deploy Prometheus and Grafana on top of Kubernetes and keep the data persistent by attaching PVC


## Pre-requisites
* Must have minikube installed
* Must have kubectl configured


### Create PVC for Prometheus and Grafana

Following is the code for creating a PVC for Prometheus

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: prom-pv
      labels:
        app: prometheus
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
          
        
Following is the code for creating a PVC for Grafana

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: grafana-pv
      labels:
        app: grafana
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
          
### Create configMap for Prometheus and Grafana

Following is the code for creating a configMap for Prometheus

    apiVersion: v1
    kind: ConfigMap 
    metadata:
      name: prom-config
    data:
      prometheus.yml: |
        global:
          scrape_interval: 10s
          evaluation_interval: 10s

        scrape_configs:
          - job_name: 'prometheus'
            static_configs:
            - targets: ['localhost:9090']    

Following is the code for creating a configMap for Grafana

    apiVersion: v1
    kind: ConfigMap 
    metadata:
      name: grafana-config
    data:
      datasource.yaml: |
        apiVersion: 1
        datasources:
          - name: Prometheus
            type: prometheus
            access: server
            url: http://192.168.99.111:31001
            version: 1
            editable: true
            
            
### Create deployment for Prometheus and Grafana

Following is the code for creating a deployment for Prometheus

    apiVersion: apps/v1 
    kind: Deployment
    metadata:
      name: prometheus
      labels:
        app: prometheus
    spec:
      selector:
        matchLabels:
          app: prometheus 
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: prometheus 
        spec:
          containers:
          - image: prom/prometheus:latest 
            name: prometheus 
            args:
            - '--config.file=/prometheus.yml'
            - '--storage.tsdb.path=/prometheus'
            ports:
            - containerPort: 9090
              name: prometheus 
            volumeMounts:
            - name: prometheus-storage
              mountPath: /prometheus 
            - name: prometheus-script 
              mountPath: /prometheus.yml
              subPath: prometheus.yml 
          volumes:
          - name: prometheus-storage
            persistentVolumeClaim:
              claimName: prom-pv
          - name: prometheus-script 
            configMap:
                name: prom-config

    

Following is the code for creating a deployment for Grafana

    apiVersion: apps/v1 
    kind: Deployment
    metadata:
      name: grafana
      labels:
        app: grafana
    spec:
      selector:
        matchLabels:
          app: grafana 
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: grafana 
        spec:
          containers:
          - image: grafana/grafana:latest 
            name: grafana 
            ports:
            - containerPort: 3000
              name: grafana 
            volumeMounts:
            - name: grafana-storage
              mountPath: /var/lib/grafana
            - name: grafana-script
              mountPath: /etc/grafana/provisioning/datasources/datasource.yaml
              subPath: datasource.yaml 
          volumes:
          - name: grafana-storage
            persistentVolumeClaim:
              claimName: grafana-pv
          - name: grafana-script
            configMap:
                name: grafana-config 


### Create a sevice for exposing Prometheus and Grafana

Following is the code for creating a service for Prometheus

    apiVersion: v1
    kind: Service
    metadata:
      name: prometheus
      labels:
        app: prometheus
    spec:
      ports:
        - port: 9090
          targetPort: 9090
          nodePort: 31001
      selector:
        app: prometheus
      type: NodePort

Following is the code for creating a service for Grafana

    apiVersion: v1
    kind: Service
    metadata:
      name: grafana
      labels:
        app: grafana
    spec:
      ports:
        - port: 3000
          targetPort: 3000
          nodePort: 31002
      selector:
        app: grafana
      type: NodePort
      
      
 Now we have two ways for deploying Prometheus and Grafana on top of Kubenetes in single go. First one is put all the code of Prometheus in one file and all the code of Grafana in one file, seprate each kind in Prometheus file as well as in Grafana file by this --- (Three hyphen). Second one is kustomization file, put all the files of Prometheus in one kustomization file and all the files of Grafana in one kustomization file and use the respective command for deploying Prometheus and Grafana on Kubernetes
 
 I am using the first way for deploying Prometheus and Grafana on top of Kubernetes
 
 Use the following command for deploying Prometheus on top of Kubernetes
 
    kubectl create -f prom-deployment.yaml
    
 Use the following command for deploying Grafana on top of Kubernetes
 
    kubectl create -f grafana-deployment.yaml
    
 Here, prom-deployment.yaml is the deployment file of Prometheus and grafana-deployment.yaml is the deployment file of Grafana
 
 ### Thank you

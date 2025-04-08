
# LAB 4.6 - Autoescalado de Pods con HPA

## Objetivos
- Configurar el Autoescalado (HPA) de Kubernetes.
- Configurar y probar el HPA para escalar automáticamente los pods basándose en la métrica de uso de CPU.

## Requerimientos
- Clúster de Kubernetes configurado.
- Metric Server habilitado en el clúster.

## Despliegue de Aplicativo

1. Crear un archivo `hpa-deploy.yaml`:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: my-hpa
      name: hpa-deploy
      namespace: lab4
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: my-hpa
      template:
        metadata:
          labels:
            app: my-hpa
        spec:
          containers:
          - name: hpa-container
            image: nginx
            ports:
            - containerPort: 80
            resources:
              requests:
                cpu: "100m"
              limits:
                cpu: "500m"
    ```

    Comandos:
    ```bash
    kubectl create -f hpa-deploy.yaml
    kubectl get pod/regular-pod-demo
    ```

2. Exponer el Deployment como Service:
    ```bash
    kubectl expose deploy hpa-deploy --name hpa-deploy-svc --port=80 -n lab4
    kubectl get svc -n lab4
    ```

## Configuración de HPA

1. Crear un archivo `hpa.yaml`:
    ```yaml
    apiVersion: autoscaling/v1
    kind: HorizontalPodAutoscaler
    metadata:
      name: hpa-example
    spec:
      maxReplicas: 3
      minReplicas: 1
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: hpa-deploy
      targetCPUUtilizationPercentage: 50
    ```

    Comandos alternativos:
    ```bash
    kubectl autoscale deployment hpa-deploy --cpu-percent=50 --min=1 --max=3 -n lab4
    kubectl create -f hpa.yaml -n lab4
    kubectl get hpa -n lab4
    ```

2. Crear carga con un pod:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: hpa-load-generator
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "while true; do wget -q -O- http://<service-ip>:80; done"]
    ```

    Comandos:
    ```bash
    kubectl create -f hpa-load-generator.yaml -n lab4
    kubectl get pods -n lab4
    ```

## Validación
- Comprobar estado del HPA:
    ```bash
    kubectl get hpa -n lab4
    kubectl get pods -n lab4
    ```

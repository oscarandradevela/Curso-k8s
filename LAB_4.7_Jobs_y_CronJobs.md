
# LAB 4.7 - Jobs y CronJobs

## Objetivos
- Crear Jobs y CronJobs en Kubernetes.

## Requerimientos
- Clúster de Kubernetes configurado.

## Creación de Jobs

1. Crear un archivo `job.yaml`:
    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: job-demo
      namespace: lab4
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["echo", "Hola, este es un ejemplo de job en ejecución"]
          restartPolicy: Never
      backoffLimit: 4
    ```

    Comandos:
    ```bash
    kubectl apply -f job.yaml
    kubectl get jobs -n lab4
    kubectl logs -l job-name=job-demo -n lab4
    ```

## Creación de CronJobs

1. Crear un archivo `cronjob.yaml`:
    ```yaml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: check-server-cronjob
      namespace: lab4
    spec:
      schedule: "* * * * *"
      successfulJobsHistoryLimit: 3
      failedJobsHistoryLimit: 1
      jobTemplate:
        metadata:
          labels:
            app: check-server
        spec:
          template:
            metadata:
              labels:
                app: check-server
            spec:
              containers:
              - name: check-server
                image: busybox
                imagePullPolicy: IfNotPresent
                command:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
              restartPolicy: OnFailure
    ```

    Comandos:
    ```bash
    kubectl apply -f cronjob.yaml
    kubectl get cronjobs -n lab4
    kubectl get pods -n lab4
    ```


To deploy a MySQL instance onto kubernetes, the most important thing you need to have is persistent volume. A storage where all data will be stored and not lost if k8s ever re-schedules the pod.

The examples are tested with GKE, so you may need to alter a few things if working with different providers.

## Things needed modification
The default values will work straight away but there might be some values you may want to change based on your needs.
1. CPU/Memory requirements - edit `deployment.yaml` and change `resources.requests`.
2. Storage size - edit `pvc.yaml` and change `resources.requests.storage`.

## Database password
By default, MySQL has been configured with `MYSQL_RANDOM_ROOT_PASSWORD` which means a random password will be generated and dumped on `stdout` during initialization. 

Once MySQL has complete its initialization process, get the logs with `kubectl logs <pod-name>` and find a string that look like this;
- `[Note] [Entrypoint]: GENERATED ROOT PASSWORD: An87dZRtqU3MBAb+RplmHUJDKScQv0Ys`

That will be your password.

## Deploying
To deploy it, apply the yaml files in order as the latter will be dependant on what is applied earlier.
1. `kubectl apply -f k8s/pvc.yaml`
2. `kubectl apply -f k8s/deployment.yaml`
3. `kubectl apply -f k8s/service.yaml`

When trying to connect to this MySQL instance, use `mysql` as the hostname.

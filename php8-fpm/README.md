1. Clone your app into `/app` folder. It will be picked up by Dockerfile when building the image.
2. Edit `configs/nginx.conf`, and set the value of `http -> server -> root`. By default it is `/var/www/html` however for many frameworks it will need to be set to `/var/www/html/public`.
3. Run build 
   - `docker build -t <registry>:<tag> .` 
     - Replace `<registry>` with whatever you want to use as registry. For dockerhub it'll be `username/reponame`. Any other would need to include the registry. Example for GCP - `gcr.io/<project_id>/<reponame>`.
     - Replace `<tag>` with a tag related to this build.
4. Push to registry
   1. `docker push <registry>:<tag>`
5. Edit `deployment.xml` and replace `<docker-image>` with `<registry>:<tag>`.
6. Add your registry's credentials to K8s.
   - https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
7. Run these two to deploy them to k8s
   1.  `kubectl apply -f deployments.yaml`
   2.  `kubectl appyl -f service.yaml`


Notes:
1. Use a proper `<tag>` value, either the version of your app or SHA sum of the last commit. K8s will cache the images so it doesn't have to re-pull the images if the pod was re-scheduled. If you use the same tag, it will not re-pull them despite actually being different from the previously used image. `imagePullPolicy: Always` does exist, it will not only slow down your pod starting up, but may incur extra bandwidth cost if your registry is outside of your k8s infrastructure.
2. The build process within Dockerfile has been set up with Laravel 9 in mind. You may need need to edit that portion if you're using a different app or framework.

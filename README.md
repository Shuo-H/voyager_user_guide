# Voyager User Guide

## Step-by-Step Guide

### Step 1: Local `pip` Installation

To install `pip` locally without root privileges:

1. Download the `get-pip.py` script:
   ```sh
   wget https://bootstrap.pypa.io/get-pip.py
   ```
   
2. Set the HOME variable to your Voyager workspace and install `pip` for the user:
   ```sh
   export HOME=/voyager/ceph/users/<your_username>
   python get-pip.py --user
   ```

3. Update your PATH to include the local `bin` directory and verify the `pip` installation:
   ```sh
   export PATH=$HOME/.local/bin:$PATH
   which pip  # Ensure pip is correctly installed in the user's path
   ```
   
4. Optionally, reset the HOME variable (if required):
   ```sh
   export HOME=/home/<your_username>
   ```

### Step 2: Load Kubernetes/Voyager Module

Load the `kubernetes/voyager` module:
```sh
module load kubernetes/voyager
```

### Step 3: Create an Interactive Job

Apply your Kubernetes pod configuration:
```sh
kubectl apply -f <filename>.yaml
```

### Step 4: Accessing the Pod

Obtain shell access to your pod:
```sh
kubectl exec -it <pod_name> -- /bin/bash  # Use "kubectl get pods" to find <pod_name>
```

### Step 5: Environment Setup

Inside the pod, set up the environment variables:
```sh
export HOME=/voyager/ceph/<your_username>
export PATH=$HOME/.local/bin:$PATH
```

### Step 6: Install Python Packages

Use `pip` to install any required packages locally:
```sh
pip install <package_name> --user
```

### Step 7: Running Tasks

- When the HOME directory is set to `/voyager/ceph/<your_username>`, models from Hugging Face and datasets will be cached under `$HOME/.cache`.
- Be aware that if your HOME directory is `/home/<your_username>`, you may exceed the maximum quota of 200GB.

### Step 8: Cleanup

After completing your tasks, exit the shell and delete the pod:
```sh
exit
kubectl delete pods <pod_name>
```

## Sample Pod Definition YAML

Here is a sample YAML configuration for creating an interactive pod:

```yaml
# Remember to set the HOME variable after gaining interactive access
apiVersion: v1
kind: Pod
metadata:
  name: bm  # Replace with your desired pod name
spec:
  securityContext: {}
  restartPolicy: Never
  volumes:
    - name: workdir
      hostPath:
        path: /voyager/ceph/users/<your_username>
        type: Directory
  containers:
    - name: gaudi-container
      image: vault.habana.ai/gaudi-docker/1.11.0/ubuntu22.04/habanalabs/pytorch-installer-2.0.1:latest
      command: ["/bin/sh", "-ec", "sleep 10800"]  # This keeps the container active for 3 hours
      resources:
        requests:
          cpu: 4
          memory: 32Gi
          habana.ai/gaudi: 1
          hugepages-2Mi: 3600Mi
        limits:
          cpu: 4
          memory: 32Gi
          habana.ai/gaudi: 1
          hugepages-2Mi: 3600Mi
      volumeMounts:
        - name: workdir
          mountPath: /voyager  # Mounting here allows easier access to the full path of "/voyager/ceph/users/<your_username>"
```

Replace `<your_username>` with your actual username, and `<filename>.yaml` and `<pod_name>` with the specific names relevant to your deployment.

--- 

Please adjust the paths and usernames to match your actual environment setup.

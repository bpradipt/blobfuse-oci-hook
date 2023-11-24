# Introduction

This is an oci hook to start Azure [blobfuse](https://github.com/Azure/azure-storage-fuse/blob/main/README.md#features).
Blobfuse allows mounting of Azure blobs on Linux. 

This oci hook is primarily meant to be used with Kata containers runtime to mount Azure blob inside the Kata VM and making the mountpoint available to the pod.

## Details

To configure the hook as a Kata prestart hook, you'll need to copy the hook binary in the oci hook directory of the Kata VM image.
The default location is `/usr/share/oci/hooks/prestart`.
Further, you'll need to add a hook configuration file (default: `/usr/share/oci/hooks/blobfuse_hookconfig.json`).

The hook expects a configuration file which is a json. An example configuration file is shown below.
```
{
  "activation_flag": "HOOK",
  "program_path": "/usr/bin/blobfuse2",
  "host_mountpoint": "/blobdata",
  "container_mountpoint": "/blobdata"
}
```

The value of the `activation_flag` (ie `HOOK` as shown above) needs to be provided as environment variable to the container.
The `container_mountpoint` can also be provided as an environment variable (`CONTAINER_MOUNTPOINT`) in the container using this hook.

Further, the blobfuse auth and other related parameters need to be provided as environment variables. Details of the blobfuse environment variables can be found [here](https://github.com/Azure/azure-storage-fuse#environment-variables).

Following is an example Kubernetes pod using this hook.
```
apiVersion: v1
kind: Pod
metadata:
  name: test
  labels:
    app: test
spec:
  runtimeClassName: kata-remote
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep"]
      args: ["infinity"]
      env:      
        - name: HOOK
          value: "true"
        - name: AZURE_STORAGE_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: storage-secret
              key: AZURE_STORAGE_ACCESS_KEY
        - name: AZURE_STORAGE_ACCOUNT_CONTAINER
          valueFrom:
            secretKeyRef:
              name: storage-secret
              key: AZURE_STORAGE_ACCOUNT_CONTAINER
        - name: AZURE_STORAGE_AUTH_TYPE
          valueFrom:
            secretKeyRef:
              name: storage-secret
              key: AZURE_STORAGE_AUTH_TYPE
           - name: AZURE_STORAGE_ACCOUNT
          valueFrom:
            secretKeyRef:
              name: storage-secret
              key: AZURE_STORAGE_ACCOUNT
        - name: AZURE_STORAGE_ACCOUNT_TYPE
          valueFrom:
            secretKeyRef:
              name: storage-secret
              key: AZURE_STORAGE_ACCOUNT_TYPE  

```
# ocp4-registry-search MachineConfig

Provide for order preference for the worker node CRIO runtime to search for and pull unqualified images from a defined list of registries.  

An unqualified image is one that *does not* include a registry spec.  `quay.io/openshift/hello-openshift:latest` is a qualified image in that it includes the `quay.io/` registry.  Whereas `openshift/hello-openshift:latest` is unqualified. 

## Implementation Notes

1. This should probably **not** be deployed to master or infra nodes.  `.metadata.labels.machineconfiguration.openshift.io/role: worker` is significant.  Be very thoughtful if you consider changing it.
2. `.spec.config.ignition.version: 3.1.0` is for OCP 4.6.x and above.  You'll need to switch to `2.1.0` for earlier versions.  The syntax is similar, if not identical.
3. The CRIO config file you're igniting into the node must be base64 encoded and that output string will replace the sample encoding in the deployment YAML.  There should be no line breaks, quotes or other characters in the encoded string.  To encode the file - `cat 88-search-registries | base64`.
4. The `88-search-registries` file should be edited to include a comma-delimited list of your registries in your preferred search order.  I would include `"docker.io"` as a final failsafe entry though I believe it's included by default.

## To deploy

```oc create -f 88-mc-search-registries.yaml```

If you `watch -n 5 oc get nodes` you'll see each of the worker nodes go into a `NotReady` state, reboot and then return to `Ready`.  The config can be confirmed as active by ssh'ing into the worker node and finding `/etc/crio/crio.conf.d/88-search-registries`.  On the worker node you can also `crictl pull some/image:tag` that does not exist in `docker.io`, but does exist in one of the other registries in your search list.


## Assistance

steve.taranto@siriuscom.com

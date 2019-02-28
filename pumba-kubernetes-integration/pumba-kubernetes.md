# How to run Pumba in Kubernetes under control of the Chaos Toolkit

[Pumba][] is a wonderful, open source tool for injecting infrastructure-level turbulent conditions into Docker-enabled environments and now, thanks to Benjamin Wilms of [Codecentric](https://www.codecentric.de/), there is a demonstrable way of integrating your [Chaos Toolkit](https://chaostoolkit.org/) automated chaos engineering experiments with all the power that [Pumba][] has to offer inside of your [Kubernetes] clusters.

[Pumba]: https://github.com/alexei-led/pumba
[Kubernetes]: https://kubernetes.io/

## Setting up Pumba in your Cluster

The way this new integration works is to rely on Kubernetes Daemon Sets to introduce Pumba into your cluster. You can configure the type of turbulent conditions you want to inject when you specify your Pumba Daemon Set.

For example, the following sample sets up Pumbas to inject a 5 second latency into a specific container:

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: pumba
  namespace: shoppingdemo
spec:
  template:
    metadata:
      labels:
        app: pumba
        type: chaos-engineering-tool
        com.gaiaadm.pumba: "true" # prevent pumba from killing itself
      name: pumba
    spec:
      containers:
        - image: gaiaadm/pumba:master
          imagePullPolicy: Always
          name: pumba
          args:
            - --interval
            - "10s"
            - netem
            - --tc-image
            - gaiadocker/iproute2
            - --duration
            - "20s"
            - delay
            - --time
            - "5000"
            - "re2:.*fashion-pod*"
          resources:
            requests:
              cpu: 10m
              memory: 5M
            limits:
              cpu: 100m
              memory: 20M
          volumeMounts:
            - name: dockersocket
              mountPath: /var/run/docker.sock
      volumes:
        - hostPath:
            path: /var/run/docker.sock
          name: dockersocket
```

> ***NOTE:*** The [full source files are also provided](sources/), along with additional examples.

## The Chaos Toolkit Experiment

With Pumba enabled in your cluster you can then write and [run](https://docs.chaostoolkit.org/reference/usage/run/) an automated [Chaos Toolkit experiment](https://docs.chaostoolkit.org/reference/api/experiment/) that triggers those turbulent conditions and explores any surfaced weaknesses detected as deviations against your system's [steady-state hypothesis](https://docs.chaostoolkit.org/reference/api/experiment/#steady-state-hypothesis):

```javascript
{
  "version": "1.0.0",
  "title": "Run Pumba under control of Chaostoolkit",
  "description": "Create Pumba as DaemonSet and afterwards, delete it",
  "tags": [
    "microservice",
    "kubernetes",
    "python"
  ],
  "steady-state-hypothesis": {
    "title": "Services are all available and healthy",
    "probes": [
    {
      "name": "api-gateway-must-still-respond",
      "provider": {
        "type": "http",
        "url": "http://gateway-master.chaos.xxx.io/startpage/cb",
        "timeout": [0.25, 0.50]
      },
      "tolerance": 200,
      "type": "probe"
    }
    ]
  },
  "method": [
  {
    "type": "probe",
    "name": "all-services-are-healthy",
    "tolerance": true,
    "provider": {
      "type": "python",
      "module": "chaosk8s.probes",
      "func": "all_microservices_healthy",
      "arguments": {
        "ns": "shoppingdemo"
      }
    }
  },
  {
    "type": "action",
    "name": "deploy_pumba_as_daemon_set",
    "provider": {
      "type": "process",
      "path": "kubectl",
      "arguments":{
        "apply":"",
        "-f":"pumba-kill-pods.yaml"
      }
    },
    "pauses": {
      "after": 5
    }
  }
  ],
  "rollbacks": [
  {
    "type": "action",
    "name": "delete_pumba_as_daemon_set",
    "provider": {
      "type": "process",
      "path": "kubectl",
      "arguments":{
        "delete":"",
        "-f":"pumba-kill-pods.yaml"
      }
    }
  }
  ]
}
```

## Summary

Thanks to the awesome project that is Pumba, and Benjamin's great contribution to the Chaos Toolkit community, you can now create [Open Chaos](https://openchaos.io/) experiments that take full advantage of all the power that Pumba has to offer.

This is another great example of how the Chaos Toolkit is flexible enough to be happily extended to integrate with the best chaos-inducing tools out there.

----

This content is licensed under the [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/) license. If you would like to submit a comment, correction or extend its ideas then please [raise an issue on the accompanying repository](https://github.com/chaosiq/chaosiq).

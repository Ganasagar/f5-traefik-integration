# f5-istio-integration
instructions for setting up f5 with Istio

1. Set the hostname for the Konvoy cluster in your cluster.yaml file as follows in spec.addons, this must match the DNS name being used to access the F5 virtual server.

```yaml
- name: konvoyconfig
  enabled: true
  values: |
    config:
      clusterHostname: ec2-34-211-7-254.us-west-2.compute.amazonaws.com
```

1. Apply the TLSProfile to be used for Istio

```yaml
apiVersion: cis.f5.com/v1
kind: TLSProfile
metadata:
  name: istio-tlsprofile
  namespace: istio-system
  labels:
    f5cr: "true"
spec:
  tls:
    # The following settings allow for passthrough TLS to be used
    clientSSL: ""
    termination: passthrough
    reference: bigip
```

2. Apply the virtual server to be used with Istio

```yaml
apiVersion: cis.f5.com/v1
kind: VirtualServer
metadata:
  name: istio-virtual-server
  namespace: istio-system
  labels:
    f5cr: "true"
spec:
  # This must be set to the IP address to be used for the virtual server
  virtualServerAddress: "10.0.128.117"
  # The port to listen on
  virtualServerHTTPSPort: 443
  # The name of the tls profile to use for the virtual server
  tlsProfileName: istio-tlsprofile
  # The name of the virtual server
  virtualServerName: istio-virtual-server
  # Do not allow http traffic
  httpTraffic: none
  pools:
  # We set the virtual server to point to the istio service on port 443
  - service: istio-ingressgateway
    servicePort: 443
```

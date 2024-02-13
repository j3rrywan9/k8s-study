# Istio Notes

## Istio ingress gateways

Istio has the concept of an ingress gateway that plays the role of the network ingress point and is responsible for guarding and controlling access to the cluster from traffic that originates outside of the cluster.
Additionally, Istio's ingress gateway handles load balancing and virtual-host routing.

If you'd like to verify that the Istio service proxy (Envoy proxy) is indeed running in the Istio ingress gateway, you can run something like this from the root directory of the book's source code:
```zsh
kubectl --context main-staging-red-usw2 -n istio-ingress exec deploy/staging-public -- ps

    PID TTY          TIME CMD
      1 ?        01:25:57 pilot-agent
     31 ?        1-13:51:07 envoy
    275 ?        00:00:00 ps
```
You should see a process listing as the output, showing the Istio service proxy command line with both `pilot-agent` and `envoy` as the running processes.
The `pilot-agent` process initially configures and bootstraps the Envoy proxy; and as we'll see in chapter 13, it implements a DNS proxy as well.

To configure Istio's ingress gateway to allow traffic into the cluster and through the service mesh, we'll start by exploring two Istio resources: `Gateway` and `VirtualService`.
Both are fundamental for getting traffic to flow in Istio, but we'll look at them only within the context of allowing traffic into the cluster.
We will cover `VirtualService` more fully in chapter 5.

All of our clusters have 3 sets of gateways.
These should almost always be `production-private`.
```zsh
kubectl --context main-staging-red-usw2 get vs -A -o yaml | yq '.items[].spec.gateways[]' | grep -oE '(production|staging|development)-(public|private)' | sort | uniq -c | sort -nk1
   1 development-public
   1 staging-public
   3 staging-private
   9 development-private
  15 production-private

kubectl --context main-production-red-usw2 get vs -A -o yaml | yq '.items[].spec.gateways[]' | grep -oE '(production|staging|development)-(public|private)' | sort | uniq -c | sort -nk1
  13 staging-public
  15 development-public
  19 production-public
  89 staging-private
 149 development-private
 166 production-private
```
There is no difference (outside of the name) for these gateways:
```zsh
diff <(kubectl -n istio-ingress get gateway production-private -o yaml | yq 'del(.metadata)') <(kubectl -n istio-ingress get gateway staging-private -o yaml | yq 'del(.metadata)')
5c5
<     istio: production-private
---
>     istio: staging-private
```
In fact Kiali recommends against setting them up this way (multiple gateways with the same exact domains listed)
https://kiali.io/docs/features/validations/#kia0301---more-than-one-gateway-for-the-same-host-port-combination

We obviously didn't know that at the time (I hope anyway).
When re-deploying Istio I didn't really have time to analyze why we did the things we did.
More -- get it up and working as fast as possible.

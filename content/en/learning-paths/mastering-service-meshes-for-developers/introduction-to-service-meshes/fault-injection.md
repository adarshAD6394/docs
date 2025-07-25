---
docType: "Chapter"
chapterTitle: "Fault Injection"
title: "Fault Injection"
description: "Meshery, collaborative Kubernetes manager"
lectures: 12
weight: 6
---

{{< chapterstyle >}}

In this chapter we will learn how to test the resiliency of an application by injecting systematic faults.
Before we start, we will need to reset the virtual services.

Using Meshery, navigate to the Istio management page:

1. Enter `default` in the `Namespace` field.
2. Click the (+) icon on the `Apply Custom Configuration` card and paste the configuration below.

Config:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - route:
        - destination:
            host: reviews
            subset: v2
```

<h2 class="chapter-sub-heading">
  Inject a route rule to create a fault using HTTP delay
</h2>

To start, we will inject a 7s delay for accessing the ratings service for a user `jason`. reviews v2 service has a 10s hard-coded connection timeout for its calls to the ratings service configured globally.

Using Meshery, navigate to the Istio management page:

1. Enter `default` in the `Namespace` field.
2. Click the (+) icon on the `Apply Custom Configuration` card and paste the configuration below.

This will update the existing virtual service definition for ratings to inject a delay for user `jason` to access the ratings V1.

In a few, we should be able to verify the virtual service by using the command below:

```sh
kubectl get virtualservice ratings -o yaml
```

Config:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
    - ratings
  http:
    - fault:
        delay:
          fixedDelay: 7s
          percentage:
            value: 100
      match:
        - headers:
            end-user:
              exact: jason
      route:
        - destination:
            host: ratings
            subset: v1
    - route:
        - destination:
            host: ratings
            subset: v1
```

Now we login to `/productpage` as user `jason` and observe that the page loads but because of the induced delay between services the reviews section will show :

<b>Error fetching product reviews!</b>
Sorry, product reviews are currently unavailable for this book.

If you logout or login as a different user, the page should load normally without any errors.

<h2 class="chapter-sub-heading">
  Inject a route rule to create a fault using HTTP abort
</h2>

In this section, , we will introduce an HTTP abort to the ratings microservices for user `jason`.

Using Meshery, navigate to the Istio management page:

1. Enter `default` in the `Namespace` field.
2. Click the (+) icon on the `Apply Custom Configuration` card and paste the configuration below.

This will update the existing virtual service definition for ratings to inject a HTTP abort for user `jason` to access the ratings V1.

In a few, we should be able to verify the virtual service by using the command below:

```sh
kubectl get virtualservice ratings -o yaml
```

Config:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
    - ratings
  http:
    - fault:
        abort:
          httpStatus: 500
          percentage:
            value: 100
      match:
        - headers:
            end-user:
              exact: jason
      route:
        - destination:
            host: ratings
            subset: v1
    - route:
        - destination:
            host: ratings
            subset: v1
```

Now we login to `/productpage` as user `jason` and observe that the page loads without any new delays but because of the induced fault between services the reviews section will show:

`Ratings service is currently unavailable`.

<h3 class="chapter-sub-heading"> Verify fault injection</h3>

Verify the fault injection by logging out (or logging in as a different user), the page should load normally without any errors.

<br />
<h3>Alternative: Manual installation</h3>
Follow these steps if the above steps did not work
<br />
<br />

<h4 class="chapter-alt-heading">Route all traffic to version V1 of all services</h4>

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

<h4 class="chapter-alt-heading">
  Route all traffic to version V2 of reviews for user Jason
</h4>

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

<h4 class="chapter-alt-heading"> Inject 7s delay for ratings service</h4>

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
```

<h4 class="chapter-alt-heading"> Inject HTTP abort for ratings service</h4>

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
```

{{< /chapterstyle >}}

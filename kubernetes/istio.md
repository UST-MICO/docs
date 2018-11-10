# Istio

Istio is a required component for Knative.

## Troubleshooting

* Istio component `istio-statsd-prom-bridge` stays in state `CrashLoopBackOff`
  -> Upgrade Istio to Istio 1.0.2 or later [Istio issue 8797](istio-statsd-prom-bridge-)

.PHONY: requirements
requirements:
	kubectl apply -f tiller/clusterrole.yaml
	kubectl create serviceaccount -n kube-system tiller
	kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
	helm init --service-account tiller
	helm init --upgrade --service-account tiller
	kubectl --namespace kube-system get pods | grep tiller
	#helm install stable/opa

ISTIO_VERSION=1.2.0
istio-$(ISTIO_VERSION):
	curl -L https://git.io/getLatestIstio | ISTIO_VERSION=$(ISTIO_VERSION) sh -


.PHONY: istio
istio: istio-$(ISTIO_VERSION)
	kubectl apply -f istio/ns.yaml
	helm template istio-$(ISTIO_VERSION)/install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -
	while [ $(shell kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l) -lt 22 ]; do echo waiting for istio; sleep 1; done
	helm template istio-$(ISTIO_VERSION)/install/kubernetes/helm/istio --name istio --namespace istio-system --values istio-$(ISTIO_VERSION)/install/kubernetes/helm/istio/values-istio-demo.yaml | kubectl apply -f -


COMPONENT := spinnaker opa
.PHONY: $(COMPONENT)
$(COMPONENT):
	kubectl apply -f $@/ns.yaml 
	helm install stable/$@ -f $@/values.yaml --name $@ --namespace $@ --timeout 1200

.PHONY: opa-policies
opa-policies:
	helm template opa/opa-policies | kubectl apply -f -

.PHONY: app-ns
app-ns:
	helm template app-ns | kubectl apply -f -
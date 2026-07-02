# 📐 Gabaritos YAML

Coleção de **templates YAML enxutos**, um para cada tipo de recurso que **não tem forma imperativa** no `kubectl` (ou cuja forma imperativa não cobre todos os campos necessários na prova).

> 🙏 **Origem:** estes templates foram adaptados do diretório [`skeletons/`](https://github.com/theplatformlab/CKA-Certified-Kubernetes-Administrator/tree/main/skeletons) do repositório [theplatformlab/CKA-Certified-Kubernetes-Administrator](https://github.com/theplatformlab/CKA-Certified-Kubernetes-Administrator), licenciado sob **MIT**. Veja o aviso completo de licença em [`../THIRD_PARTY_NOTICES.md`](../THIRD_PARTY_NOTICES.md).

---

## 🎯 Filosofia de uso

O nosso [banco de exercícios](../simulados/BANCO-DE-EXERCICIOS.md) já ensina que a forma imperativa do `kubectl` é sempre a mais rápida. Mas alguns recursos **não têm** subcomando imperativo (ou têm um muito limitado). Para esses casos, use o fluxo:

```bash
# 1. Tente gerar via imperativo + $do primeiro (mesmo que incompleto)
k create <recurso> ... $do > meu-recurso.yaml

# 2. Se não existir forma imperativa, comece a partir do gabarito daqui
cp gabaritos-yaml/networkpolicy.yaml meu-recurso.yaml

# 3. Edite apenas os campos necessários e aplique
k apply -f meu-recurso.yaml
```

> 💡 **Não decore YAML inteiro.** Decore a *estrutura* (quais campos existem e onde) e pratique editar rápido com `vim`/`nano`. Ter estes gabaritos "na cabeça" é mais rápido do que navegar pela documentação para cada campo.

---

## 📄 Arquivos Disponíveis

| Arquivo | Recurso | Por que não tem (bom) atalho imperativo |
| :--- | :--- | :--- |
| [`pod.yaml`](pod.yaml) | Pod com `resources` | `kubectl run` não aceita `--requests`/`--limits` |
| [`deployment.yaml`](deployment.yaml) | Deployment | Base rápida com `resources` já preenchido |
| [`daemonset.yaml`](daemonset.yaml) | DaemonSet | Sem subcomando `kubectl create daemonset` |
| [`statefulset.yaml`](statefulset.yaml) | StatefulSet | Sem subcomando `kubectl create statefulset` |
| [`job.yaml`](job.yaml) / [`cronjob.yaml`](cronjob.yaml) | Job / CronJob | Base de referência para campos avançados |
| [`configmap-secret.yaml`](configmap-secret.yaml) | ConfigMap + Secret | Útil quando o dado não é um literal simples |
| [`service.yaml`](service.yaml) | Service | Referência para `type`, `selector`, `ports` |
| [`ingress.yaml`](ingress.yaml) | Ingress | Regras de path/host mais complexas que o `--rule` |
| [`gateway-api.yaml`](gateway-api.yaml) | Gateway + HTTPRoute | Gateway API é 100% declarativa |
| [`networkpolicy.yaml`](networkpolicy.yaml) | NetworkPolicy | **Sem forma imperativa** — sempre YAML |
| [`pv.yaml`](pv.yaml) / [`pvc.yaml`](pvc.yaml) | PersistentVolume / Claim | PV **sem forma imperativa** |
| [`storageclass.yaml`](storageclass.yaml) | StorageClass | **Sem forma imperativa** |
| [`rbac.yaml`](rbac.yaml) / [`clusterrole.yaml`](clusterrole.yaml) | Role/RoleBinding, ClusterRole | Formas imperativas existem mas são limitadas para regras compostas |
| [`serviceaccount.yaml`](serviceaccount.yaml) | ServiceAccount | Referência mínima |
| [`hpa.yaml`](hpa.yaml) | HorizontalPodAutoscaler | `kubectl autoscale` não cobre métricas customizadas |
| [`resourcequota.yaml`](resourcequota.yaml) / [`limitrange.yaml`](limitrange.yaml) | ResourceQuota / LimitRange | **Sem forma imperativa** |
| [`securitycontext.yaml`](securitycontext.yaml) | SecurityContext (Pod/Container) | Referência de campos de segurança |
| [`sidecar-init-container.yaml`](sidecar-init-container.yaml) | initContainers / sidecar | Padrão multi-container comum na prova |
| [`validatingadmissionpolicy.yaml`](validatingadmissionpolicy.yaml) | ValidatingAdmissionPolicy | **Sem forma imperativa** |

---

Pratique aplicando estes gabaritos nos exercícios do [`../simulados/BANCO-DE-EXERCICIOS.md`](../simulados/BANCO-DE-EXERCICIOS.md) sempre que a solução pedir um YAML do zero. ☸️

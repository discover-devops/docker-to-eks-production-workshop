# Kubernetes Practice Questions

This is a practice sheet, not an answer sheet.

Every question has a short pointer telling you what to think about and what to search
for. Do the searching yourself, write your own answer, and prove it on a real cluster
wherever you can. The answer sheet comes later.

How to get value out of this:

1. Write your answer before you search.
2. Then search, and correct what you got wrong.
3. Then prove it with kubectl on a real cluster.
4. Mark every question where your first answer was wrong. Those are your revision list.

Sections run in dependency order. The scenario questions at the end are only answerable
if you have worked through the sections before them.

---

## Section 1. Fundamentals

**1. What problem does Kubernetes solve that Docker on a single host does not?**
Direction: scheduling across machines, surviving host failure, health based routing.

**2. What is the difference between imperative and declarative infrastructure?**
Direction: telling a machine what to do versus recording what should be true.

**3. What is a reconciliation loop?**
Direction: desired state, actual state, continuous comparison, act on the difference.

**4. Name four Kubernetes features that are all the same reconciliation loop underneath.**
Direction: self healing, scaling, rolling updates, rescheduling.

**5. What is the difference between kubectl create and kubectl apply?**
Direction: create versus declare. Search for idempotency and last applied configuration.

**6. What does kubectl apply do on the second run with an unchanged file?**
Direction: nothing. Explain why that matters for a pipeline.

**7. What is a Kubernetes object versus a controller?**
Direction: the record of intent versus the program that acts on it.

**8. What is the API server and why does everything talk to it?**
Direction: single entry point, authentication, authorisation, validation, persistence.

**9. What are the four parts of every Kubernetes manifest?**
Direction: apiVersion, kind, metadata, spec. Also status and who writes it.

**10. What is the difference between spec and status in an object?**
Direction: what you want versus what is. Who owns each.

**11. What does kubectl explain do and why is it underused?**
Direction: try kubectl explain deployment.spec.strategy.

**12. What is the difference between kubectl apply dry run client and server?**
Direction: local schema check versus full API server validation.

**13. Why did Kubernetes win over Docker Swarm and Mesos?**
Direction: vendor neutral governance, cloud provider adoption, ecosystem effects.

**14. What is the Kubernetes API group and version, and why do they change?**
Direction: v1, apps/v1, networking.k8s.io/v1. Search for alpha, beta, GA graduation.

---

## Section 2. Control plane

**15. Name the control plane components and give each one job.**
Direction: api server, etcd, scheduler, controller manager, cloud controller manager.

**16. What does etcd store and what happens if you lose it?**
Direction: cluster state. Think about backups and why managed control planes are attractive.

**17. What does the scheduler actually do, step by step?**
Direction: filtering then scoring. Search for predicates and priorities, or filter and score plugins.

**18. What is the controller manager and how many controllers does it run?**
Direction: one loop per resource type. Name five controllers.

**19. What does the cloud controller manager do?**
Direction: load balancers, node lifecycle, routes. Why it was split out of the controller manager.

**20. How does a Deployment become running pods? Trace the full path.**
Direction: apply, api server, etcd, deployment controller, replicaset controller, scheduler, kubelet.

**21. Which component assigns a pod to a node, and which one starts the container?**
Direction: two different components. Be precise.

**22. What happens if the control plane goes down while pods are running?**
Direction: existing pods keep running. What stops working.

**23. What is the difference between a managed control plane and a self managed one?**
Direction: who patches etcd at 2 AM. Cost and access trade-offs.

---

## Section 3. Node components

**24. What runs on every worker node?**
Direction: kubelet, kube-proxy, container runtime.

**25. What does the kubelet do?**
Direction: watches for pods assigned to its node, runs probes, reports status.

**26. What does kube-proxy do and how does it do it?**
Direction: Service routing. Search for iptables mode and IPVS mode.

**27. Why is there no central proxy in Kubernetes networking?**
Direction: every node routes independently. Think about bottlenecks and failure domains.

**28. What is containerd doing on a Kubernetes node and where did Docker Engine go?**
Direction: search for CRI, dockershim removal, OCI images.

**29. What is a static pod and where does it come from?**
Direction: kubelet reads a directory on disk. Search for staticPodPath.

**30. What is a node condition and what conditions exist?**
Direction: Ready, MemoryPressure, DiskPressure, PIDPressure.

**31. What happens when a node goes NotReady?**
Direction: taints, eviction timers, pod rescheduling. Search for node lifecycle controller.

**32. How do you safely take a node out of service for maintenance?**
Direction: cordon and drain. Know the difference.

**33. What does kubectl cordon do that drain does not?**
Direction: one stops new pods, the other also removes existing ones.

---

## Section 4. Pods

**34. Why does Kubernetes schedule pods rather than containers?**
Direction: shared network namespace, shared storage, atomic scheduling.

**35. What do containers in the same pod share?**
Direction: IP address, network namespace, volumes, lifecycle.

**36. How do two containers in the same pod talk to each other?**
Direction: localhost. Think about port conflicts.

**37. What is a sidecar container and give three real uses.**
Direction: log shipper, proxy, metrics exporter, config reloader.

**38. What is an init container and how is it different from a sidecar?**
Direction: runs to completion before the main containers start.

**39. Can init containers run in parallel?**
Direction: no. Explain the ordering guarantee.

**40. What is a native sidecar in newer Kubernetes versions?**
Direction: search for restartPolicy Always on an init container.

**41. What is an ephemeral container and when do you use one?**
Direction: debugging a running pod, especially a distroless one. Search for kubectl debug.

**42. Does a pod get its own IP address? Who assigns it?**
Direction: the CNI plugin. On EKS, think about where that address comes from.

**43. Are pods repaired or replaced when they fail?**
Direction: replaced. Explain the consequence for anything addressing them.

**44. What are the pod phases and what does each mean?**
Direction: Pending, Running, Succeeded, Failed, Unknown.

**45. What is the difference between a pod phase and a container state?**
Direction: Running, Waiting, Terminated, and the reasons attached to each.

**46. What is CrashLoopBackOff and what is Kubernetes actually doing?**
Direction: exponential backoff on restart. It is a symptom, not a cause.

**47. What is ImagePullBackOff and list four causes.**
Direction: wrong name, private registry, no credentials, rate limit, wrong architecture.

**48. What happens step by step when you delete a pod?**
Direction: SIGTERM, grace period, SIGKILL. Search for terminationGracePeriodSeconds.

**49. What is a preStop hook and why would you need one?**
Direction: draining connections before shutdown. Think about load balancer deregistration timing.

**50. What is the pod restart policy and what values does it take?**
Direction: Always, OnFailure, Never. Which controllers allow which.

**51. Why should you almost never create a bare pod?**
Direction: nothing is watching it. Test it by deleting one.

---

## Section 5. Labels, selectors and annotations

**52. What is the difference between a label and an annotation?**
Direction: selectable identifying metadata versus non identifying data.

**53. How does a ReplicaSet know which pods belong to it?**
Direction: a label query, not a list. Think about the consequence.

**54. What happens if you manually create a pod with labels matching a ReplicaSet selector?**
Direction: try it. The ReplicaSet counts it.

**55. What is the difference between matchLabels and matchExpressions?**
Direction: equality versus set based selectors. Search for In, NotIn, Exists.

**56. Why is a Deployment selector immutable after creation?**
Direction: think about orphaned ReplicaSets and pods.

**57. Name five recommended standard labels.**
Direction: search for app.kubernetes.io recommended labels.

**58. Give three real uses of annotations.**
Direction: ingress controller configuration, checksums to trigger restarts, tooling metadata.

**59. How do you select pods across multiple label values in one query?**
Direction: kubectl get pods -l 'app in (a,b)'.

---

## Section 6. Workload controllers

**60. What is the relationship between Deployment, ReplicaSet and Pod?**
Direction: which one is version aware and which one just counts.

**61. Why does a Deployment create a new ReplicaSet when you change the image?**
Direction: one ReplicaSet per pod template. Think about what enables rollback.

**62. What does revisionHistoryLimit control?**
Direction: how many old ReplicaSets are kept. Trade-off with rollback depth.

**63. When would you use a StatefulSet instead of a Deployment?**
Direction: stable identity, stable storage, ordered operations.

**64. What guarantees does a StatefulSet give that a Deployment does not?**
Direction: stable pod names, ordinal index, ordered start and stop, per pod PVC.

**65. What is a headless Service and why do StatefulSets need one?**
Direction: clusterIP None, per pod DNS records.

**66. What is a DaemonSet and give three real uses.**
Direction: one pod per node. Log agents, CNI plugins, node exporters.

**67. How does a DaemonSet handle a new node joining the cluster?**
Direction: think about the reconciliation loop again.

**68. What is the difference between a Job and a CronJob?**
Direction: run to completion versus scheduled run to completion.

**69. What do completions and parallelism control in a Job?**
Direction: how many must succeed and how many run at once.

**70. What is backoffLimit and what happens when it is exceeded?**
Direction: retries then failure.

**71. What is concurrencyPolicy in a CronJob?**
Direction: Allow, Forbid, Replace. When each matters.

**72. What is a ReplicationController and why should you not use it?**
Direction: the predecessor to ReplicaSet. Know it exists for legacy clusters.

**73. What is a custom resource and a custom controller?**
Direction: CRD plus a controller. Search for the operator pattern.

---

## Section 7. Probes and lifecycle

**74. What are the three probe types and what does each control?**
Direction: startup, readiness, liveness. One sentence each.

**75. What is the single most important difference between liveness and readiness?**
Direction: one restarts, one reroutes. Say which is which without looking.

**76. What happens when a readiness probe fails?**
Direction: removed from Service endpoints, pod keeps running.

**77. What happens when a liveness probe fails?**
Direction: container killed and restarted.

**78. What goes wrong if you use a liveness probe where a readiness probe belongs?**
Direction: a dependency outage becomes a cluster wide restart storm.

**79. When do you need a startup probe?**
Direction: slow starting applications. What does it suspend while running.

**80. What are the probe handler types?**
Direction: httpGet, tcpSocket, exec, grpc.

**81. Explain initialDelaySeconds, periodSeconds, timeoutSeconds, failureThreshold, successThreshold.**
Direction: write each in one line and then compute how long a failure takes to be detected.

**82. Should a health endpoint check the database? Argue both sides.**
Direction: cascading failure versus false healthy. Think about liveness versus readiness separately.

**83. A pod is Running but not Ready. What does that mean and what is affected?**
Direction: readiness gating and Service endpoints.

**84. How do probes interact with a rolling update?**
Direction: readiness is what makes maxUnavailable zero meaningful.

---

## Section 8. Resources and QoS

**85. What is the difference between a resource request and a resource limit?**
Direction: scheduling reservation versus enforced ceiling.

**86. Which one does the scheduler use?**
Direction: only one of them. Be precise.

**87. What happens if you omit requests entirely?**
Direction: the scheduler assumes zero and packs the node. Explain the outcome.

**88. What happens when a container exceeds its memory limit?**
Direction: OOM kill. Search for exit code 137.

**89. What happens when a container exceeds its CPU limit?**
Direction: throttling, not killing. Explain why CPU and memory behave differently.

**90. What are the three QoS classes and how is each assigned?**
Direction: Guaranteed, Burstable, BestEffort. Search for the exact rules.

**91. Which pods get evicted first under node memory pressure?**
Direction: QoS class ordering.

**92. What is a LimitRange and what problem does it solve?**
Direction: defaults and bounds per namespace.

**93. What is a ResourceQuota and how is it different from a LimitRange?**
Direction: namespace total versus per object.

**94. Is it good practice to set CPU limits? Argue both sides.**
Direction: search for CPU throttling debates and CFS quota behaviour.

**95. What are ephemeral storage requests and limits?**
Direction: node disk consumed by logs and writable layers.

---

## Section 9. Scheduling

**96. How does the scheduler choose a node? Name the two phases.**
Direction: filter then score.

**97. What is nodeSelector and what are its limits?**
Direction: simple equality. Why affinity replaced it for complex cases.

**98. What is the difference between requiredDuringScheduling and preferredDuringScheduling?**
Direction: hard rule versus soft preference.

**99. What is pod affinity and pod anti affinity? Give a real use for each.**
Direction: co-locate for latency, spread for availability.

**100. What is topologyKey and why does it matter for anti affinity?**
Direction: hostname versus zone. Think about what failure you are protecting against.

**101. What are taints and tolerations?**
Direction: a node repels pods unless the pod tolerates it.

**102. What are the three taint effects?**
Direction: NoSchedule, PreferNoSchedule, NoExecute. What is different about the third.

**103. Give a real use case for a taint.**
Direction: GPU nodes, dedicated nodes, control plane nodes.

**104. What is topology spread constraint and how is it different from anti affinity?**
Direction: skew control across zones or nodes, with more control than anti affinity.

**105. What is a PriorityClass and what is preemption?**
Direction: high priority pods evicting lower priority ones when capacity is short.

**106. What is a PodDisruptionBudget and what does it protect against?**
Direction: voluntary disruptions like drains. Not node crashes.

**107. Why does a node drain sometimes hang?**
Direction: PDBs, unmanaged pods, local storage.

**108. What is nodeName and why should you avoid setting it?**
Direction: bypasses the scheduler entirely.

**109. A pod is Pending forever. Name six possible reasons.**
Direction: insufficient resources, no matching node, taints, node selector, PVC unbound, pod IP exhaustion.

---

## Section 10. Services

**110. What is a Service and what three things does it give you?**
Direction: stable virtual IP, DNS name, label selector.

**111. Is a ClusterIP assigned to any machine?**
Direction: no. Explain what actually implements it.

**112. What is an EndpointSlice and what determines membership?**
Direction: pod IPs of matching pods. The condition for inclusion matters.

**113. What is the difference between Endpoints and EndpointSlices?**
Direction: search for why EndpointSlices replaced Endpoints at scale.

**114. Name the four Service types and when you would use each.**
Direction: ClusterIP, NodePort, LoadBalancer, ExternalName.

**115. How do NodePort and LoadBalancer relate to ClusterIP?**
Direction: they stack. Each builds on the previous one.

**116. What port range does NodePort use?**
Direction: 30000 to 32767 by default.

**117. What is the cost problem with Service type LoadBalancer at scale?**
Direction: one cloud load balancer per Service. Count for forty microservices.

**118. What is a headless Service and what does DNS return for one?**
Direction: clusterIP None. Pod IPs directly.

**119. When do you want a headless Service?**
Direction: StatefulSets, gRPC clients, client side load balancing.

**120. Is Service load balancing per request or per connection?**
Direction: this surprises people with keep alive and gRPC. Search for connection pinning.

**121. What is sessionAffinity and what values does it take?**
Direction: None and ClientIP.

**122. What is externalTrafficPolicy and what does Local do?**
Direction: preserves client source IP but can cause imbalance.

**123. What is targetPort and why use a named port instead of a number?**
Direction: the Service follows a container port rename without editing.

**124. Your Service has no endpoints. What are the two most likely causes?**
Direction: selector mismatch, or no pod is Ready.

**125. What is an ExternalName Service used for?**
Direction: mapping a cluster local name to an external DNS name like an RDS endpoint.

---

## Section 11. DNS and cluster networking

**126. What is the fully qualified DNS name of a Service?**
Direction: service.namespace.svc.cluster.local.

**127. Why does the short name work within a namespace?**
Direction: look at the search path in a pod resolv.conf.

**128. How do you reach a Service in another namespace?**
Direction: include the namespace in the name.

**129. What runs the cluster DNS and how many replicas does it usually have?**
Direction: CoreDNS in kube-system.

**130. What is dnsPolicy and what values exist?**
Direction: ClusterFirst, Default, None, ClusterFirstWithHostNet.

**131. What are the four networking rules Kubernetes requires of any CNI?**
Direction: search for the Kubernetes network model. Pod to pod without NAT is one of them.

**132. What does a CNI plugin do?**
Direction: assigns pod IPs and wires the network namespace.

**133. What is different about the AWS VPC CNI compared with an overlay network?**
Direction: pods get real VPC IP addresses. Consequences for security groups and IP exhaustion.

**134. How can you run out of IP addresses on EKS and how do you prevent it?**
Direction: pods per node limits, subnet sizing, prefix delegation.

**135. What is a NetworkPolicy and what is the default behaviour without one?**
Direction: allow all. Explain what changes once one policy selects a pod.

**136. How do you write a default deny NetworkPolicy?**
Direction: empty podSelector with policyTypes.

**137. Does the AWS VPC CNI enforce NetworkPolicy out of the box?**
Direction: check the version. Search for VPC CNI network policy support and Calico as an alternative.

**138. What is the difference between ingress and egress rules in a NetworkPolicy?**
Direction: direction of the connection, not the response.

---

## Section 12. Ingress

**139. What is the difference between an Ingress and an Ingress controller?**
Direction: a document versus a program. What happens if you apply one without the other.

**140. Name three Ingress controllers and how they differ.**
Direction: NGINX runs pods, AWS Load Balancer Controller provisions an ALB, Traefik does its own thing.

**141. What is ingressClassName for?**
Direction: selecting which controller acts on this Ingress.

**142. What are the three pathTypes and how do they match?**
Direction: Prefix, Exact, ImplementationSpecific.

**143. Does an Ingress rewrite the request path before it reaches the pod?**
Direction: no by default. What tools can rewrite and which cannot.

**144. How do you do host based routing with an Ingress?**
Direction: the host field in the rule.

**145. How is TLS configured on an Ingress?**
Direction: two ways. Secret with a certificate, or a cloud certificate by annotation.

**146. What is the difference between target-type ip and instance on the AWS controller?**
Direction: pod IPs in the target group versus nodes on a NodePort. Count the hops.

**147. What is an Ingress group and what problem does it solve?**
Direction: many Ingress objects sharing one load balancer.

**148. How does a pod failing its readiness probe affect an ALB target group?**
Direction: deregistration. Trace who tells the ALB.

**149. What is the Gateway API and why does it exist?**
Direction: role separation, expressiveness, portability. Is it replacing Ingress today.

**150. Ingress ADDRESS stays empty. What do you check first?**
Direction: controller logs, subnet tags, IAM permissions, IngressClass.

---

## Section 13. Configuration and secrets

**151. What is a ConfigMap and what are the three ways to consume one?**
Direction: environment variables, envFrom, mounted volume.

**152. What happens to a mounted ConfigMap when you update it?**
Direction: eventually updated in the volume. What about environment variables.

**153. Why do environment variables from a ConfigMap not update on change?**
Direction: they are set at process start.

**154. How do you force pods to restart when a ConfigMap changes?**
Direction: annotation with a checksum, or a rolling restart.

**155. Are Kubernetes Secrets encrypted?**
Direction: base64 is not encryption. Search for encryption at rest and KMS providers.

**156. Who can read a Secret in a namespace by default?**
Direction: think about RBAC and service accounts.

**157. Name three better alternatives to plain Kubernetes Secrets on AWS.**
Direction: Secrets Manager with the CSI driver, External Secrets Operator, IRSA with direct SDK calls.

**158. What is the Secrets Store CSI driver and how does it work?**
Direction: mounts external secrets as files. Search for the AWS provider.

**159. What is the difference between a Secret and a ConfigMap technically?**
Direction: less than people think. Search for the actual differences in storage and handling.

**160. What is an imagePullSecret and when do you need one?**
Direction: private registries. How EKS avoids needing one for ECR.

**161. Why should configuration come from the environment rather than the image?**
Direction: one image for every environment. Search for twelve factor config.

---

## Section 14. Storage

**162. What is the difference between a volume, a PersistentVolume and a PersistentVolumeClaim?**
Direction: pod scoped, cluster resource, and the request for one.

**163. What is emptyDir and when does its data disappear?**
Direction: pod lifetime, not container lifetime. Test it.

**164. Why do you need emptyDir at /tmp with a read only root filesystem?**
Direction: something still needs to write.

**165. What is a StorageClass and what is dynamic provisioning?**
Direction: a template for creating volumes on demand.

**166. What are the access modes and what do they really mean?**
Direction: ReadWriteOnce, ReadOnlyMany, ReadWriteMany, ReadWriteOncePod.

**167. Why can most block storage not do ReadWriteMany?**
Direction: think about attaching one EBS volume to two instances. What can do it.

**168. What is the reclaim policy and what values exist?**
Direction: Retain, Delete. Which is safer and which is the default.

**169. What is CSI and why does it exist?**
Direction: out of tree storage drivers. Search for in-tree plugin removal.

**170. On EKS, what does the EBS CSI driver do and why is it a separate add-on?**
Direction: dynamic EBS provisioning. It is not installed by default.

**171. Why does an EBS backed pod fail to reschedule to another availability zone?**
Direction: EBS is zonal. Search for volume topology constraints.

**172. When would you use EFS instead of EBS on EKS?**
Direction: shared writable storage across pods and zones.

**173. Should you run a database in Kubernetes? Argue both sides.**
Direction: operators, StatefulSets, storage guarantees versus managed services.

---

## Section 15. Security

**174. What is the difference between authentication and authorisation in Kubernetes?**
Direction: who you are versus what you may do. On EKS, which system does each.

**175. What is RBAC and what are its four object types?**
Direction: Role, ClusterRole, RoleBinding, ClusterRoleBinding.

**176. What is the difference between a Role and a ClusterRole?**
Direction: namespace scoped versus cluster scoped. When a ClusterRole is used in a RoleBinding.

**177. How do you check whether you can perform an action?**
Direction: kubectl auth can-i. Also with the as flag for impersonation.

**178. What is a ServiceAccount and what does a pod get by default?**
Direction: every pod gets one. Search for automountServiceAccountToken.

**179. Why should you disable token automounting for pods that do not call the API?**
Direction: reduce what a compromised pod can do.

**180. What is IRSA and what problem does it solve?**
Direction: per pod IAM credentials via OIDC instead of node role permissions.

**181. Why is giving the node role broad AWS permissions a bad idea?**
Direction: every pod on the node inherits them.

**182. What is EKS Pod Identity and how does it differ from IRSA?**
Direction: newer mechanism, no OIDC trust policy per cluster. Search for the comparison.

**183. What is a securityContext and name six fields you would set?**
Direction: runAsNonRoot, runAsUser, readOnlyRootFilesystem, allowPrivilegeEscalation, capabilities, seccompProfile.

**184. What is the difference between a pod level and a container level securityContext?**
Direction: scope and which wins on conflict.

**185. What does runAsNonRoot actually check, and why does a username in the image fail it?**
Direction: the kubelet must resolve a UID before starting the container.

**186. What is fsGroup and when do you need it?**
Direction: volume ownership for a non root process.

**187. What replaced PodSecurityPolicy?**
Direction: Pod Security Admission. Search for the three levels.

**188. Name the three Pod Security Standards levels and what each enforces.**
Direction: privileged, baseline, restricted.

**189. What is an admission controller and what are the two webhook types?**
Direction: validating and mutating. When each runs.

**190. Give a real use for a mutating admission webhook.**
Direction: sidecar injection, default labels, image rewriting.

**191. What are policy engines like OPA Gatekeeper and Kyverno used for?**
Direction: enforcing organisational rules at admission time.

**192. What is the risk of hostNetwork, hostPID and hostPath?**
Direction: each removes an isolation boundary. Name what each exposes.

**193. Why is a container escape more damaging on a shared node?**
Direction: shared kernel and what else is running there.

**194. How do you audit who did what in a cluster?**
Direction: audit logs. On EKS, where do they go.

**195. What is a seccomp profile and what does RuntimeDefault give you?**
Direction: syscall filtering.

---

## Section 16. Namespaces and multi-tenancy

**196. What does a namespace actually isolate and what does it not?**
Direction: names and scoped objects. It is not a security boundary by itself.

**197. Which resources are not namespaced?**
Direction: nodes, PersistentVolumes, ClusterRoles, StorageClasses. Find the full list.

**198. How do you enforce fair resource use between teams sharing a cluster?**
Direction: ResourceQuota plus LimitRange plus RBAC.

**199. What do you add to make a namespace closer to a real tenancy boundary?**
Direction: NetworkPolicy, RBAC, quotas, Pod Security Admission, node isolation.

**200. When would you use separate clusters instead of separate namespaces?**
Direction: blast radius, compliance, version skew, noisy neighbours.

---

## Section 17. Autoscaling

**201. What are the three main autoscaling mechanisms and what does each scale?**
Direction: HPA scales pods, VPA scales pod size, Cluster Autoscaler scales nodes.

**202. What does HPA need in order to work at all?**
Direction: metrics server, and resource requests set on the pods.

**203. Why does HPA not work if requests are missing?**
Direction: utilisation is a percentage of something.

**204. Can HPA scale on custom or external metrics?**
Direction: yes. Search for the custom metrics API and KEDA.

**205. Why should you not run HPA and VPA on the same metric?**
Direction: they fight each other.

**206. What is Cluster Autoscaler and what triggers it?**
Direction: pending pods that cannot be scheduled.

**207. What is Karpenter and how does it differ from Cluster Autoscaler?**
Direction: provisions right sized nodes directly rather than scaling node groups.

**208. Why is pod scaling faster than node scaling? Give the rough numbers.**
Direction: seconds versus minutes. Explain why.

**209. How do you scale to handle a traffic spike that is faster than node provisioning?**
Direction: headroom, overprovisioning with low priority placeholder pods.

---

## Section 18. Rollouts and updates

**210. What are maxSurge and maxUnavailable and what does each control?**
Direction: above and below the replica count during an update.

**211. What does maxSurge 1 with maxUnavailable 0 guarantee?**
Direction: capacity never drops. What must exist for this to be safe.

**212. What is the Recreate strategy and when is it correct?**
Direction: full stop then start. Think about incompatible schema changes.

**213. How do you roll back a Deployment and what makes it fast?**
Direction: the old ReplicaSet still exists at zero replicas.

**214. What does kubectl rollout status actually wait for?**
Direction: readiness of the new pods.

**215. How do you pause and resume a rollout, and why would you?**
Direction: canary style manual checkpoints.

**216. How would you do a canary release with plain Kubernetes objects?**
Direction: two Deployments, one Service, replica ratio. Then explain the limits.

**217. How would you do a blue green release?**
Direction: two Deployments, switch the Service selector.

**218. What tools do progressive delivery properly and what do they add?**
Direction: Argo Rollouts, Flagger. Metric analysis and automatic rollback.

**219. Why does kubectl set image work but is a bad production practice?**
Direction: the cluster no longer matches Git. Search for configuration drift and GitOps.

**220. What is GitOps and what does it change about deployment?**
Direction: Git as the source of truth, a controller pulls. Search for Argo CD and Flux.

---

## Section 19. Observability

**221. Where do container logs go and what happens when a pod is deleted?**
Direction: stdout, node disk, rotation. Then explain why you need a log shipper.

**222. What does kubectl logs with the previous flag give you?**
Direction: the crashed container's output. Essential for CrashLoopBackOff.

**223. What are Kubernetes Events and how long are they kept?**
Direction: default retention is short. Search for the event TTL.

**224. Why should kubectl get events be your first troubleshooting command?**
Direction: it is the narrative of what the controllers did.

**225. What is metrics-server and what depends on it?**
Direction: kubectl top and HPA.

**226. What is the difference between metrics-server and Prometheus?**
Direction: short lived resource metrics for autoscaling versus a full metrics system.

**227. What are the four golden signals?**
Direction: latency, traffic, errors, saturation.

**228. How do you debug a pod with no shell in the image?**
Direction: kubectl debug with an ephemeral container.

**229. How do you see resource usage per pod and per node?**
Direction: kubectl top pods and kubectl top nodes.

---

## Section 20. EKS specifics

**230. What does AWS manage in EKS and what do you manage?**
Direction: draw the line precisely. Control plane, nodes, workloads, networking.

**231. What are the node group options and when would you use each?**
Direction: managed node groups, self managed, Fargate.

**232. What is underneath a managed node group?**
Direction: an Auto Scaling Group and a launch template.

**233. What are the trade-offs of Fargate on EKS?**
Direction: no node management, per pod cost, and several real limitations. Find them.

**234. What are EKS add-ons and name four common ones.**
Direction: VPC CNI, CoreDNS, kube-proxy, EBS CSI driver, Pod Identity agent.

**235. How does authentication work on EKS and what changed with access entries?**
Direction: aws-auth ConfigMap historically, EKS access entries now.

**236. Why does kubectl work for the cluster creator but not for a colleague?**
Direction: the creating IAM principal gets admin automatically.

**237. What does withOIDC enable and what needs it?**
Direction: IRSA. Which controllers depend on it.

**238. Which subnet tags does the AWS Load Balancer Controller look for?**
Direction: kubernetes.io/role/elb and internal-elb.

**239. Why does an ALB require subnets in at least two availability zones?**
Direction: AWS enforces it. Explain the resilience reason.

**240. How do you upgrade an EKS cluster safely?**
Direction: control plane first, then add-ons, then node groups. Search for version skew policy.

**241. What is the version skew policy between control plane and kubelet?**
Direction: how many minor versions behind is allowed.

**242. How do you give a pod permission to read an S3 bucket, correctly?**
Direction: IRSA or Pod Identity, not the node role.

**243. What costs money in an EKS cluster? List everything.**
Direction: control plane hourly, nodes, EBS volumes, NAT gateway, load balancers, data transfer.

**244. Why does eksctl delete cluster sometimes fail on the VPC?**
Direction: load balancers or security groups created by controllers still exist.

---

## Section 21. Helm and packaging

**245. What problem does Helm solve?**
Direction: templating, packaging, versioned releases of a set of manifests.

**246. What are a chart, a release and a repository?**
Direction: one line each.

**247. What is values.yaml and how does override precedence work?**
Direction: defaults, then values files, then set flags.

**248. What does helm upgrade with install do?**
Direction: idempotent install or upgrade. Useful in pipelines.

**249. How do you see what Helm will actually apply?**
Direction: helm template, and the dry run flag.

**250. What is a Helm hook and give a use case?**
Direction: pre-install jobs, database migrations.

**251. How is Kustomize different from Helm?**
Direction: overlays and patches versus templating. When each fits.

---

## Section 22. Troubleshooting

**252. A pod is Pending. Walk through your diagnosis.**
Direction: describe the pod, read Events, check requests, taints, PVCs, node capacity.

**253. A pod is in CrashLoopBackOff. Walk through your diagnosis.**
Direction: logs with previous, exit code, config, probes, permissions.

**254. A pod is ImagePullBackOff. Walk through your diagnosis.**
Direction: describe pod for the exact registry error. Name, credentials, architecture, rate limit.

**255. A pod shows CreateContainerConfigError. What are the likely causes?**
Direction: missing ConfigMap or Secret, or a runAsNonRoot violation.

**256. A pod is Running but the Service returns 503.**
Direction: no ready endpoints, selector mismatch, wrong targetPort.

**257. A Service has endpoints but connections time out.**
Direction: NetworkPolicy, security groups, app binding to localhost.

**258. DNS resolution fails inside a pod.**
Direction: CoreDNS pods, resolv.conf, namespace name form, NetworkPolicy blocking DNS.

**259. Pods restart every few minutes with no application errors.**
Direction: liveness probe too aggressive, or OOM kills. Check exit code 137.

**260. A node is NotReady. What do you check?**
Direction: kubelet status, node conditions, disk pressure, network path to the API server.

**261. kubectl commands are slow or timing out.**
Direction: API server load, network path, large result sets, admission webhooks.

**262. A rolling update is stuck at partway complete.**
Direction: new pods are not becoming Ready. Then treat it as a readiness problem.

**263. A pod cannot mount its PVC.**
Direction: unbound claim, zone mismatch, missing CSI driver, access mode.

**264. Two teams say the cluster is slow. How do you find out who is causing it?**
Direction: kubectl top, resource quotas, QoS classes, noisy neighbours.

**265. An admission webhook is failing and nothing can be deployed.**
Direction: failurePolicy. Search for how a webhook can lock you out of your own cluster.

**266. You applied a manifest and nothing happened. No error, no pod.**
Direction: wrong namespace, wrong context, selector mismatch, or it was rejected silently.

---

## Section 23. Scenario based

**267. Your application needs to reach another service in the cluster. Compare hardcoding a pod IP, using a Service name, and using an Ingress. When is each right?**
Direction: mortality of pods, internal versus external traffic.

**268. A pod is healthy according to Kubernetes but customers report errors. How is that possible?**
Direction: what the health endpoint actually checks.

**269. A service must not lose a single request during deployment. What do you configure?**
Direction: readiness probe, maxUnavailable zero, preStop hook, graceful shutdown, PDB.

**270. Your cluster runs out of pod IP addresses on EKS. Explain and fix.**
Direction: VPC CNI IP allocation, subnet size, prefix delegation, instance type limits.

**271. A team wants each microservice exposed publicly. There are forty services. Design it.**
Direction: one Ingress with path or host routing versus forty LoadBalancer Services. Cost and DNS.

**272. Half your pods are on one node and it fails, taking the service down. What did you fail to configure?**
Direction: topology spread constraints or anti affinity.

**273. A batch job must run nightly and never overlap with the previous run. Design it.**
Direction: CronJob with concurrencyPolicy Forbid, plus backoffLimit and history limits.

**274. A pod needs to read a secret from AWS Secrets Manager. Give two designs and pick one.**
Direction: Secrets Store CSI driver, External Secrets Operator, or direct SDK with IRSA.

**275. A deployment succeeded but the new version has a bug in production. What are your options and how fast is each?**
Direction: rollout undo, redeploy previous tag, and why Git should be the source of truth.

**276. Your organisation wants to stop anyone deploying images from Docker Hub. How do you enforce it?**
Direction: admission policy with Kyverno or Gatekeeper, image registry allow list.

**277. A developer says their pod cannot reach the internet. Everything looks fine in Kubernetes.**
Direction: NAT gateway, route tables, security groups, NetworkPolicy egress.

**278. You must prove which image digest was running in production last Tuesday. Can you?**
Direction: deploy by digest, audit logs, GitOps history, immutable tags.

**279. A cluster upgrade is planned. Write the sequence and the checks between each step.**
Direction: control plane, add-ons, node groups, PDBs, deprecated API versions.

**280. A deprecated API version is removed in the next Kubernetes release and your manifests use it.**
Direction: search for kubent or pluto. Then explain the upgrade sequence.

**281. Design the security posture for a namespace running a payments service. List ten controls.**
Direction: RBAC, service account scoping, NetworkPolicy, Pod Security Admission, securityContext,
image policy, secrets management, quotas, audit logging, node isolation.

**282. Your team uses kubectl apply from laptops to deploy. Describe five risks and the fix.**
Direction: no review, no audit, drift, credentials on laptops, no rollback record. GitOps.

**283. One microservice is being overwhelmed by another during traffic spikes. What Kubernetes level controls help?**
Direction: resource limits, HPA, PDB, rate limiting at the mesh or ingress, priority classes.

**284. Compare running one large cluster versus several small ones for an organisation of twelve teams.**
Direction: blast radius, cost, operational overhead, isolation, upgrade coordination.

**285. Explain to a developer why their pod IP keeps changing and what they should use instead.**
Direction: pods are replaced not repaired. Service DNS.

---

## Section 24. Boundaries and design judgement

**286. What does Kubernetes not solve that people expect it to?**
Direction: application design, data consistency, cost control, developer experience.

**287. When is Kubernetes the wrong choice?**
Direction: single small service, tiny team, no operational capacity. Argue it honestly.

**288. What is the operational cost of running Kubernetes that teams underestimate?**
Direction: upgrades, add-on lifecycle, security patching, expertise, on call.

**289. Compare EKS, ECS and plain EC2 with Auto Scaling for a five service application.**
Direction: be specific about what each gives and what each costs in effort.

**290. A manager asks why the same application needs Kubernetes when EC2 with an Auto Scaling Group already self heals. Answer properly.**
Direction: speed of replacement, density, deployment time, rollback, health based routing.
Concede what the ASG genuinely does well.

**291. What is a service mesh and what problems does it solve that Kubernetes does not?**
Direction: mTLS, retries, circuit breaking, fine grained traffic splitting, observability.

**292. Do you need a service mesh? Argue both sides.**
Direction: complexity cost versus what you actually need. Search for when a mesh is overkill.

**293. List the ten things you would check in a Kubernetes manifest code review.**
Direction: image tag, resources, probes, securityContext, labels, replicas, strategy,
namespace, secrets handling, PDB.

**294. Write down the five metrics you would alert on for a production service.**
Direction: error rate, latency, pod restarts, ready replica count, saturation.

**295. If you had to explain the entire value of Kubernetes in three sentences, what would you say?**
Direction: this is the summary question. Write it, then compare with what you wrote on day one.

---

## How to practise

Take one or two sections a day. For every question, write your answer first, then search,
then prove it with kubectl on a live cluster. A question you can answer but cannot
demonstrate is a question you do not yet know.

The scenario sections are the interview questions. They are not answerable by memorisation,
only by understanding the sections above them.

Bring your wrong answers to the next session.

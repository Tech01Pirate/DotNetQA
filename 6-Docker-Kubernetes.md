# Docker & Kubernetes

## Question 126-175: Docker & K8s Essentials

**Q126: Docker image layers**
- Each RUN, COPY, ADD creates layer
- Layers are stacked, cached, reused
- Smaller base image + fewer layers = faster builds

**Q127: ENTRYPOINT vs CMD**
- ENTRYPOINT: Always executes (configurable with arguments)
- CMD: Default arguments, can be overridden
- Example: `ENTRYPOINT ["java"] CMD ["-jar", "app.jar"]`

**Q128: Multi-stage builds**
```dockerfile
FROM node:16 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM node:16-alpine
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/index.js"]
# Result: Smaller image (only production files)
```

**Q129: Container security**
- Run as non-root: `USER app`
- Use read-only filesystem: `--read-only`
- Scan images for vulnerabilities: `docker scan image:tag`
- No secrets in image: Use environment variables

**Q130: Pod lifecycle**
- Pending → Running → Succeeded/Failed
- Init containers run first
- Readiness probe: Ready for traffic?
- Liveness probe: Healthy? Restart if not

**Q131: Deployment vs StatefulSet**
- Deployment: Stateless, any pod can replace
- StatefulSet: Stateful, persistent identity, ordered scaling

**Q132: Service types**
- ClusterIP: Internal only
- NodePort: External via port
- LoadBalancer: Cloud load balancer
- ExternalName: Map to external DNS

**Q133: Ingress controller**
Routes external HTTP/HTTPS to services

**Q134: ConfigMap vs Secret**
- ConfigMap: Non-sensitive config (can be plain text)
- Secret: Passwords, tokens (base64 encoded, at rest encryption)

**Q135: Liveness vs readiness**
- Liveness: Is pod alive? Restart if fails
- Readiness: Should receive traffic? Remove from endpoints if fails

**Q136: HPA (Horizontal Pod Autoscaler)**
Scales pod count based on CPU/memory

**Q137: Rolling update**
Gradually replace old pods with new ones, zero downtime

**Q138: Blue-green vs canary**
- Blue-green: Switch all traffic at once
- Canary: Route percentage to new version

**Q139: Pod crashloop debugging**
- Check logs: `kubectl logs pod-name`
- Check events: `kubectl describe pod pod-name`
- Check resources: Might be OOM

**Q140: Resource requests/limits**
- Request: Minimum guaranteed
- Limit: Maximum allowed

**Q141: Node affinity**
Pin pods to specific nodes

**Q142: Taints/tolerations**
Prevent pods from scheduling on nodes

**Q143: Persistent volumes**
Storage that survives pod restart

**Q144: Helm basics**
Package manager for K8s, templating

**Q145: AKS upgrades**
Update cluster version without downtime

**Q146: Observability stack**
- Prometheus: Metrics
- Grafana: Visualization
- ELK/Loki: Logs
- Jaeger: Traces

**Q147: Service mesh**
Istio/Linkerd handle service-to-service communication, security, observability

**Q148: Secret rotation**
Regularly update secrets, applications reload without restart

**Q149: Cost optimization**
- Spot instances (preemptible)
- Reserve instances
- Right-size resources
- Cluster autoscaler

**Q150: Production AKS architecture**
```
Internet → Azure Front Door (WAF)
         → Application Gateway
         → AKS Ingress Controller
         → Service Mesh (Istio)
         → Microservices (Deployments)
         → Persistent Volumes (managed disks)
         → Azure Container Registry (images)
```


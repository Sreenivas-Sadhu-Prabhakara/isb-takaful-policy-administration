# Takaful Policy Administration

BIAN Service Domain microservice — part of the [islamic-platform](../../islamic-platform/) landscape.

| | |
|---|---|
| **Business Area** | Takaful and Shared Operations |
| **Business Domain** | Takaful |
| **Functional Pattern** | Administer |
| **Asset Type** | Takaful Certificate |
| **Control Record** | Takaful Certificate Administrative Plan |
| **K8s Namespace** | `isb-operations` |
| **Stack** | Java 21 · Spring Boot 3 · Resilience4j · Cilium mesh |

> ⚠️ **Phase 1 (shallow):** real REST API over an in-memory store. Phase 2 replaces the store with per-domain persistence and real domain logic. This repo was stamped from `islamic-platform/generator` — regenerate rather than hand-editing boilerplate.

## BIAN Semantic API

| Method | Path | BIAN action term |
|---|---|---|
| GET | `/v1/service-domain` | — (SD metadata) |
| POST | `/v1/takaful-certificate-administrative-plan/initiate` | Initiate |
| GET | `/v1/takaful-certificate-administrative-plan` | Retrieve (list) |
| GET | `/v1/takaful-certificate-administrative-plan/{crId}/retrieve` | Retrieve |
| PUT | `/v1/takaful-certificate-administrative-plan/{crId}/update` | Update |
| PUT | `/v1/takaful-certificate-administrative-plan/{crId}/control` | Control — body `{"action": "suspend"\|"resume"\|"terminate"}` |

OpenAPI UI: `/swagger-ui.html` · Health: `/actuator/health` · Metrics: `/actuator/prometheus`

**API contract:** [`api/openapi.yaml`](api/openapi.yaml) — owned by **this repo** (contract-per-repo; no central contracts repo). The runtime spec at `/v3/api-docs` must stay compatible with it; Phase 2 adds contract tests that enforce this.

## Run locally

```bash
mvn spring-boot:run
curl localhost:8080/v1/service-domain

# lifecycle smoke test
curl -X POST localhost:8080/v1/takaful-certificate-administrative-plan/initiate -H 'content-type: application/json' -d '{"note":"hello"}'
```

## Build & containerize

```bash
mvn -B verify
docker build -t isb/isb-takaful-policy-administration:0.1.0 .
```

## Deploy (Helm → K8s with Cilium mesh)

```bash
helm upgrade --install isb-takaful-policy-administration ./helm -n isb-operations
```

Exposed through the platform Gateway at path prefix `/isb-takaful-policy-administration` (Cilium Gateway API). Mesh policy (`CiliumNetworkPolicy`) allows: gateway ingress, same-area peers, Prometheus — everything else denied.

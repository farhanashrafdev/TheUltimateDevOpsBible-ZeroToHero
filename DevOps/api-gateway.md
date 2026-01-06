# API Gateway: Complete Guide

## 🎯 Introduction

An API Gateway is a server that acts as a single entry point for all client requests to your backend services. It handles routing, composition, protocol translation, and cross-cutting concerns like authentication, rate limiting, and monitoring.

### Why API Gateway?

```
Without API Gateway:
┌────────┐     ┌─────────────────┐
│ Client │────▶│ Service A       │
│        │────▶│ Service B       │
│        │────▶│ Service C       │
└────────┘     └─────────────────┘
Problems: Direct exposure, no central auth, scattered rate limiting

With API Gateway:
┌────────┐     ┌─────────────┐     ┌─────────────────┐
│ Client │────▶│ API Gateway │────▶│ Service A       │
│        │     │             │────▶│ Service B       │
│        │     │             │────▶│ Service C       │
└────────┘     └─────────────┘     └─────────────────┘
Benefits: Single entry, centralized auth, rate limiting, monitoring
```

## 🔧 Popular API Gateways

### Comparison Matrix

```yaml
┌────────────────┬──────────┬────────────┬───────────┬───────────┐
│ Gateway        │ Type     │ Best For   │ Learning  │ Cost      │
├────────────────┼──────────┼────────────┼───────────┼───────────┤
│ Kong           │ OSS/Ent  │ K8s, APIs  │ Medium    │ Free/Paid │
│ NGINX          │ OSS/Ent  │ General    │ Easy      │ Free/Paid │
│ Traefik        │ OSS/Ent  │ K8s, Auto  │ Easy      │ Free/Paid │
│ AWS API GW     │ Managed  │ AWS        │ Easy      │ Pay/use   │
│ Istio Gateway  │ OSS      │ Service Mesh│ Hard     │ Free      │
│ Ambassador     │ OSS/Ent  │ K8s native │ Medium    │ Free/Paid │
│ Apigee         │ Managed  │ Enterprise │ Medium    │ Expensive │
└────────────────┴──────────┴────────────┴───────────┴───────────┘
```

## 🦍 Kong Gateway

### Installation

```bash
# Kubernetes with Helm
helm repo add kong https://charts.konghq.com
helm repo update

# Install Kong with DB-less mode
helm install kong kong/kong \
  --namespace kong --create-namespace \
  --set ingressController.installCRDs=false \
  --set admin.enabled=true \
  --set admin.http.enabled=true

# Verify installation
kubectl get pods -n kong
kubectl get svc -n kong
```

### Kong Configuration

```yaml
# Kong Ingress Resource
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  annotations:
    konghq.com/strip-path: "true"
    konghq.com/plugins: "rate-limiting,jwt-auth"
spec:
  ingressClassName: kong
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /users
        pathType: Prefix
        backend:
          service:
            name: users-service
            port:
              number: 80
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: orders-service
            port:
              number: 80
---
# Rate Limiting Plugin
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: rate-limiting
config:
  minute: 100
  hour: 1000
  policy: local
plugin: rate-limiting
---
# JWT Authentication Plugin
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: jwt-auth
plugin: jwt
config:
  claims_to_verify:
    - exp
```

### Kong Plugins

```yaml
# Request Transformer
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: request-transformer
plugin: request-transformer
config:
  add:
    headers:
      - "X-Request-ID:$(uuid)"
    querystring:
      - "version:v1"
  remove:
    headers:
      - "X-Internal-Header"
---
# Response Transformer
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: response-transformer
plugin: response-transformer
config:
  add:
    headers:
      - "X-Cache-Status:HIT"
  remove:
    headers:
      - "X-Powered-By"
---
# Correlation ID
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: correlation-id
plugin: correlation-id
config:
  header_name: X-Correlation-ID
  generator: uuid
  echo_downstream: true
---
# IP Restriction
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: ip-restriction
plugin: ip-restriction
config:
  allow:
    - 10.0.0.0/8
    - 192.168.0.0/16
```

## 🔵 Traefik

### Installation

```bash
# Kubernetes with Helm
helm repo add traefik https://helm.traefik.io/traefik
helm repo update

helm install traefik traefik/traefik \
  --namespace traefik --create-namespace \
  --set dashboard.enabled=true \
  --set dashboard.domain=traefik.example.com
```

### Traefik Configuration

```yaml
# IngressRoute
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: api-routes
  namespace: production
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`api.example.com`) && PathPrefix(`/users`)
      kind: Rule
      services:
        - name: users-service
          port: 80
      middlewares:
        - name: rate-limit
        - name: auth-headers
    - match: Host(`api.example.com`) && PathPrefix(`/orders`)
      kind: Rule
      services:
        - name: orders-service
          port: 80
  tls:
    certResolver: letsencrypt
---
# Rate Limit Middleware
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: rate-limit
spec:
  rateLimit:
    average: 100
    burst: 50
    period: 1m
---
# Strip Prefix Middleware
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: strip-api-prefix
spec:
  stripPrefix:
    prefixes:
      - /api
---
# Basic Auth Middleware
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: basic-auth
spec:
  basicAuth:
    secret: auth-secret
---
# Retry Middleware
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: retry-middleware
spec:
  retry:
    attempts: 3
    initialInterval: 100ms
---
# Circuit Breaker Middleware
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: circuit-breaker
spec:
  circuitBreaker:
    expression: LatencyAtQuantileMS(50.0) > 100 || ResponseCodeRatio(500, 600, 0, 600) > 0.3
```

### Traefik with Let's Encrypt

```yaml
# Certificate Resolver
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: secure-route
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`api.example.com`)
      kind: Rule
      services:
        - name: api-service
          port: 80
  tls:
    certResolver: letsencrypt
---
# Traefik Helm Values for Let's Encrypt
# values.yaml
additionalArguments:
  - "--certificatesresolvers.letsencrypt.acme.email=admin@example.com"
  - "--certificatesresolvers.letsencrypt.acme.storage=/data/acme.json"
  - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
```

## ☁️ AWS API Gateway

### REST API

```yaml
# Serverless Framework Configuration
service: my-api

provider:
  name: aws
  runtime: python3.9
  stage: ${opt:stage, 'dev'}
  region: us-east-1

functions:
  getUsers:
    handler: handlers.get_users
    events:
      - http:
          path: /users
          method: get
          cors: true
          authorizer:
            type: COGNITO_USER_POOLS
            authorizerId: !Ref ApiAuthorizer
            
  createUser:
    handler: handlers.create_user
    events:
      - http:
          path: /users
          method: post
          cors: true
          request:
            schemas:
              application/json: ${file(schemas/create-user.json)}

resources:
  Resources:
    ApiAuthorizer:
      Type: AWS::ApiGateway::Authorizer
      Properties:
        Name: CognitoAuthorizer
        Type: COGNITO_USER_POOLS
        IdentitySource: method.request.header.Authorization
        RestApiId: !Ref ApiGatewayRestApi
        ProviderARNs:
          - !GetAtt UserPool.Arn
```

### HTTP API (v2)

```hcl
# Terraform - AWS HTTP API
resource "aws_apigatewayv2_api" "main" {
  name          = "main-api"
  protocol_type = "HTTP"
  
  cors_configuration {
    allow_headers = ["*"]
    allow_methods = ["GET", "POST", "PUT", "DELETE"]
    allow_origins = ["https://app.example.com"]
    max_age       = 300
  }
}

resource "aws_apigatewayv2_stage" "prod" {
  api_id      = aws_apigatewayv2_api.main.id
  name        = "prod"
  auto_deploy = true
  
  access_log_settings {
    destination_arn = aws_cloudwatch_log_group.api_logs.arn
    format = jsonencode({
      requestId      = "$context.requestId"
      ip             = "$context.identity.sourceIp"
      requestTime    = "$context.requestTime"
      httpMethod     = "$context.httpMethod"
      routeKey       = "$context.routeKey"
      status         = "$context.status"
      responseLength = "$context.responseLength"
      latency        = "$context.responseLatency"
    })
  }
  
  default_route_settings {
    throttling_burst_limit = 1000
    throttling_rate_limit  = 500
  }
}

# Lambda Integration
resource "aws_apigatewayv2_integration" "users" {
  api_id             = aws_apigatewayv2_api.main.id
  integration_type   = "AWS_PROXY"
  integration_uri    = aws_lambda_function.users.invoke_arn
  integration_method = "POST"
}

resource "aws_apigatewayv2_route" "get_users" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "GET /users"
  target    = "integrations/${aws_apigatewayv2_integration.users.id}"
  
  authorization_type = "JWT"
  authorizer_id      = aws_apigatewayv2_authorizer.jwt.id
}

# JWT Authorizer
resource "aws_apigatewayv2_authorizer" "jwt" {
  api_id           = aws_apigatewayv2_api.main.id
  authorizer_type  = "JWT"
  identity_sources = ["$request.header.Authorization"]
  name             = "jwt-authorizer"
  
  jwt_configuration {
    audience = ["api-client"]
    issuer   = "https://cognito-idp.us-east-1.amazonaws.com/${aws_cognito_user_pool.main.id}"
  }
}
```

## 🛡️ Common Gateway Features

### Rate Limiting Implementation

```yaml
# NGINX rate limiting
http {
  # Define rate limit zone
  limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
  limit_req_zone $http_x_api_key zone=key_limit:10m rate=100r/s;
  
  server {
    location /api/ {
      # Apply rate limits
      limit_req zone=api_limit burst=20 nodelay;
      limit_req zone=key_limit burst=50;
      
      # Custom error for rate limiting
      limit_req_status 429;
      
      proxy_pass http://backend;
    }
  }
}
```

```python
# Custom Rate Limiter with Redis
import redis
import time
from functools import wraps
from flask import request, jsonify

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def rate_limit(max_requests: int, window_seconds: int):
    """Token bucket rate limiter using Redis"""
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            # Get client identifier
            client_id = request.headers.get('X-API-Key') or request.remote_addr
            key = f"rate_limit:{client_id}"
            
            # Sliding window implementation
            now = time.time()
            window_start = now - window_seconds
            
            pipe = redis_client.pipeline()
            # Remove old entries
            pipe.zremrangebyscore(key, 0, window_start)
            # Count current window
            pipe.zcard(key)
            # Add current request
            pipe.zadd(key, {str(now): now})
            # Set expiry
            pipe.expire(key, window_seconds)
            
            results = pipe.execute()
            request_count = results[1]
            
            if request_count >= max_requests:
                return jsonify({
                    "error": "Rate limit exceeded",
                    "retry_after": window_seconds
                }), 429
            
            response = f(*args, **kwargs)
            
            # Add rate limit headers
            response.headers['X-RateLimit-Limit'] = str(max_requests)
            response.headers['X-RateLimit-Remaining'] = str(max_requests - request_count - 1)
            response.headers['X-RateLimit-Reset'] = str(int(now + window_seconds))
            
            return response
        return wrapper
    return decorator

@app.route('/api/users')
@rate_limit(max_requests=100, window_seconds=60)
def get_users():
    return jsonify({"users": []})
```

### Authentication Patterns

```yaml
# Kong - OAuth2 Plugin
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: oauth2
plugin: oauth2
config:
  scopes:
    - read
    - write
    - admin
  mandatory_scope: true
  enable_authorization_code: true
  enable_client_credentials: true
  token_expiration: 3600
---
# Kong - OpenID Connect
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: oidc
plugin: openid-connect
config:
  issuer: "https://auth.example.com/.well-known/openid-configuration"
  client_id: "api-gateway"
  client_secret:
    - "${{OIDC_CLIENT_SECRET}}"
  auth_methods:
    - bearer
  scopes_required:
    - api:read
```

### Request/Response Transformation

```yaml
# Traefik - Header Modifications
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: headers-middleware
spec:
  headers:
    customRequestHeaders:
      X-Forwarded-Proto: "https"
      X-Real-IP: "true"
    customResponseHeaders:
      X-Frame-Options: "SAMEORIGIN"
      X-Content-Type-Options: "nosniff"
      Strict-Transport-Security: "max-age=31536000; includeSubDomains"
      Content-Security-Policy: "default-src 'self'"
    accessControlAllowMethods:
      - GET
      - POST
      - PUT
      - DELETE
    accessControlAllowOriginList:
      - "https://app.example.com"
    accessControlMaxAge: 100
```

### Circuit Breaker Pattern

```yaml
# Istio - Circuit Breaker
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: users-service
spec:
  host: users-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: UPGRADE
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      minHealthPercent: 30
```

## 📊 Monitoring & Observability

### Prometheus Metrics

```yaml
# Kong Prometheus Plugin
apiVersion: configuration.konghq.com/v1
kind: KongClusterPlugin
metadata:
  name: prometheus
  labels:
    global: "true"
plugin: prometheus
config:
  per_consumer: true
  status_code_metrics: true
  latency_metrics: true
  bandwidth_metrics: true
  upstream_health_metrics: true
```

### Key Metrics to Monitor

```yaml
Essential Gateway Metrics:
  Request Metrics:
    - request_count (by route, status code)
    - request_latency (p50, p95, p99)
    - request_size
    - response_size
  
  Error Metrics:
    - error_count (4xx, 5xx)
    - timeout_count
    - circuit_breaker_trips
  
  Resource Metrics:
    - connections_active
    - connections_total
    - upstream_health
  
  Business Metrics:
    - requests_by_api_key
    - rate_limit_hits
    - auth_failures
```

### Grafana Dashboard

```json
{
  "dashboard": {
    "title": "API Gateway Dashboard",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(kong_http_requests_total[5m])) by (route)",
            "legendFormat": "{{route}}"
          }
        ]
      },
      {
        "title": "Latency P99",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.99, sum(rate(kong_latency_bucket[5m])) by (le, route))",
            "legendFormat": "{{route}}"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(kong_http_requests_total{code=~\"5..\"}[5m])) / sum(rate(kong_http_requests_total[5m])) * 100",
            "legendFormat": "Error %"
          }
        ]
      }
    ]
  }
}
```

## ✅ Best Practices

### Security
- [ ] Enable TLS/HTTPS everywhere
- [ ] Implement authentication (OAuth2, JWT, API keys)
- [ ] Configure rate limiting per client
- [ ] Enable WAF for common attacks
- [ ] Use IP allowlisting for internal APIs
- [ ] Rotate API keys regularly

### Performance
- [ ] Enable response caching where appropriate
- [ ] Configure connection pooling
- [ ] Set appropriate timeouts
- [ ] Enable compression (gzip/brotli)
- [ ] Use keep-alive connections

### Reliability
- [ ] Implement circuit breakers
- [ ] Configure health checks
- [ ] Set up retry policies with exponential backoff
- [ ] Enable request/response buffering
- [ ] Plan for graceful degradation

### Observability
- [ ] Export metrics to Prometheus
- [ ] Enable access logging
- [ ] Add correlation IDs
- [ ] Set up alerting on key metrics
- [ ] Create dashboards for visibility

---

**Next Steps**:
- Learn [Service Mesh](./service-mesh.md)
- Explore [Kubernetes Networking](./kubernetes-networking.md)
- Master [Monitoring & Observability](./monitoring-observability.md)



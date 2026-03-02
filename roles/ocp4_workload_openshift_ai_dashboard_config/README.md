# ocp4_workload_openshift_ai_dashboard_config

## Description

This role configures the OpenShift AI Dashboard, including:
- Dashboard settings and features
- Notebook controller configuration
- GPU hardware profiles
- GatewayConfig cookie settings for authentication

## Requirements

- OpenShift AI operator must be installed
- `kubernetes.core` collection

## Role Variables

### Dashboard Settings

```yaml
ocp4_workload_openshift_ai_dashboard_config_settings:
  disableTracking: false
  disableModelRegistry: false
  disableModelCatalog: false
  disableKServeMetrics: false
  genAiStudio: true
  modelAsService: true
  disableLMEval: false
```

### Notebook Controller

```yaml
ocp4_workload_openshift_ai_dashboard_config_notebook_controller:
  enabled: true
  pvcSize: 20Gi
```

### Hardware Profiles

```yaml
ocp4_workload_openshift_ai_hardware_profile_gpu_identifiers:
- defaultCount: '1'
  displayName: CPU
  identifier: cpu
  maxCount: '8'
  minCount: 1
  resourceType: CPU
- defaultCount: 12Gi
  displayName: Memory
  identifier: memory
  maxCount: 16Gi
  minCount: 1Gi
  resourceType: Memory
- defaultCount: 1
  displayName: GPU
  identifier: nvidia.com/gpu
  maxCount: 4
  minCount: 1
  resourceType: Accelerator
```

### GatewayConfig Cookie Settings

**IMPORTANT**: These settings prevent authentication redirect loops when using Keycloak.

```yaml
# Cookie expiry time - MUST be less than Keycloak SSO Session Idle timeout
# Default Keycloak session idle is 5 minutes (300 seconds)
ocp4_workload_openshift_ai_dashboard_config_gateway_cookie_expire: 4m

# Cookie refresh interval - how often to refresh the cookie during user activity
# Should be 1/4 to 1/3 of the expire time
ocp4_workload_openshift_ai_dashboard_config_gateway_cookie_refresh: 1m
```

#### Cookie Timing Explanation

The GatewayConfig cookie settings must be coordinated with Keycloak session timeouts to prevent redirect loops:

| Setting | Recommended Value | Reason |
|---------|------------------|--------|
| Cookie Expire | `4m` (240 seconds) | Must be < Keycloak SSO Session Idle (5 min) |
| Cookie Refresh | `1m` (60 seconds) | Keeps active users logged in by refreshing cookie |

**How it works**:
- If cookie expires **after** Keycloak session → redirect loop
- If cookie expires **before** Keycloak session → clean re-authentication
- Cookie refresh ensures active users stay logged in

**Formula**: `Cookie Refresh < Cookie Expire < Keycloak Session Idle`

Example: `60s < 240s < 300s` ✅

## Dependencies

- OpenShift AI operator must be installed and running

## Example Playbook

```yaml
- name: Configure OpenShift AI Dashboard
  hosts: localhost
  roles:
    - role: rhpds.openshift_ai.ocp4_workload_openshift_ai_dashboard_config
      vars:
        ocp4_workload_openshift_ai_dashboard_config_gateway_cookie_expire: 4m
        ocp4_workload_openshift_ai_dashboard_config_gateway_cookie_refresh: 1m
        ocp4_workload_openshift_ai_dashboard_config_settings:
          genAiStudio: true
          modelAsService: true
```

## License

Apache 2.0

## Author Information

Red Hat GPTE

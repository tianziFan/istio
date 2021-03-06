# RBAC

Istio RBAC (Role Based Access Control) defines ServiceRole and ServiceRoleBinding objects.

A ServiceRole specification includes a list of rules (permissions). Each rule has the following standard fields: * services: a list of services. * methods: HTTP methods or gRPC methods. Note that gRPC methods should be presented in the form of “packageName.serviceName/methodName”. * paths: HTTP paths. It is ignored in gRPC case.

In addition to the standard fields, operators can use custom fields in the “constraints” section. The name of a custom field must match one of the “properties” in the “action” part of the “authorization” template (https://github.com/istio/istio/blob/master/mixer/template/authorization/template.proto).

For example, suppose we define an instance of the “authorization” template, named “requestcontext”.

```yaml
apiVersion: "config.istio.io/v1alpha1"
kind: authorization
metadata:
  name: requestcontext
  namespace: istio-system
spec:
  subject:
    user: request.auth.principal | ""
    groups: request.auth.principal | ""
    properties:
      service: source.service | ""
      namespace: source.namespace | ""
  action:
    namespace: destination.namespace | ""
    service: destination.service | ""
    method: request.method | ""
    path: request.path | ""
    properties:
      version: request.headers["version"] | ""
```

Below is an example of ServiceRole object “product-viewer”, which has “read” (“GET” and “HEAD”) access to “products.svc.cluster.local” service at versions “v1” and “v2”. “path” is not specified, so it applies to any path in the service.

```yaml
apiVersion: "config.istio.io/v1alpha1"
kind: ServiceRole
metadata:
  name: products-viewer
  namespace: default
spec:
  rules:
  - services: ["products.svc.cluster.local"]
    methods: ["GET", "HEAD"]
    constraints:
    - key: "version"
      value: ["v1", "v2"]
```

A ServiceRoleBinding specification includes two parts: * “roleRef” refers to a ServiceRole object in the same namespace. * A list of “subjects” that are assigned the roles.

A subject is represented with a set of “properties”. The name of a property must match one of the fields (“user” or “groups” or one of the “properties”) in the “subject” part of the “authorization” template (https://github.com/istio/istio/blob/master/mixer/template/authorization/template.proto).

Below is an example of ServiceRoleBinding object “test-binding-products”, which binds two subjects to ServiceRole “product-viewer”: * User “alice@yahoo.com” * “reviews” service in “abc” namespace.

```
apiVersion: "config.istio.io/v1alpha1"
kind: ServiceRoleBinding
metadata:
  name: test-binding-products
  namespace: default
spec:
  subjects:
  - user: alice@yahoo.com
  - properties:
      service: "reviews"
      namespace: "abc"
  roleRef:
    kind: ServiceRole
    name: "products-viewer"
```

## AccessRule

AccessRule defines a permission to access a list of services.

| Field         | Type                      | Description                                                  |
| ------------- | ------------------------- | ------------------------------------------------------------ |
| `services`    | `string[]`                | Required. A list of service names. Exact match, prefix match, and suffix match are supported for service names. For example, the service name “bookstore.mtv.cluster.local” matches “bookstore.mtv.cluster.local” (exact match), or “bookstore*” (prefix match), or “*.mtv.cluster.local” (suffix match). If set to [“*”], it refers to all services in the namespace. |
| `paths`       | `string[]`                | Optional. A list of HTTP paths. Exact match, prefix match, and suffix match are supported for paths. For example, the path “/books/review” matches “/books/review” (exact match), or “/books/*” (prefix match), or “*/review” (suffix match). If not specified, it applies to any path. |
| `methods`     | `string[]`                | Required. A list of HTTP methods (e.g., “GET”, “POST”) or gRPC methods. gRPC methods must be presented as fully-qualified name in the form of packageName.serviceName/methodName. If set to [“*”], it applies to any method. |
| `constraints` | `AccessRule.Constraint[]` | Optional. Extra constraints in the ServiceRole specification. The above ServiceRole examples shows an example of constraint “version”. |

## AccessRule.Constraint

Definition of a custom constraint. The key of a custom constraint must match one of the “properties” in the “action” part of the “authorization” template (https://github.com/istio/istio/blob/master/mixer/template/authorization/template.proto).

| Field    | Type       | Description                                                  |
| -------- | ---------- | ------------------------------------------------------------ |
| `key`    | `string`   | Key of the constraint.                                       |
| `values` | `string[]` | List of valid values for the constraint. Exact match, prefix match, and suffix match are supported for constraint values. For example, the value “v1alpha2” matches “v1alpha2” (exact match), or “v1*” (prefix match), or “*alpha2” (suffix match). |

## RoleRef

RoleRef refers to a role object.

| Field  | Type     | Description                                                  |
| ------ | -------- | ------------------------------------------------------------ |
| `kind` | `string` | Required. The type of the role being referenced. Currently, “ServiceRole” is the only supported value for “kind”. |
| `name` | `string` | Required. The name of the ServiceRole object being referenced. The ServiceRole object must be in the same namespace as the ServiceRoleBinding object. |

## ServiceRole

ServiceRole specification contains a list of access rules (permissions). This represent the “Spec” part of the ServiceRole object. The name and namespace of the ServiceRole is specified in “metadata” section of the ServiceRole object.

| Field   | Type           | Description                                                  |
| ------- | -------------- | ------------------------------------------------------------ |
| `rules` | `AccessRule[]` | Required. The set of access rules (permissions) that the role has. |

## ServiceRoleBinding

ServiceRoleBinding assigns a ServiceRole to a list of subjects. This represents the “Spec” part of the ServiceRoleBinding object. The name and namespace of the ServiceRoleBinding is specified in “metadata” section of the ServiceRoleBinding object.

| Field      | Type        | Description                                                  |
| ---------- | ----------- | ------------------------------------------------------------ |
| `subjects` | `Subject[]` | Required. List of subjects that are assigned the ServiceRole object. |
| `roleRef`  | `RoleRef`   | Required. Reference to the ServiceRole object.               |

## Subject

Subject defines an identity or a group of identities. The identity is either a user or a group or identified by a set of “properties”. The name of the “properties” must match the “properties” in the “subject” part of the “authorization” template (https://github.com/istio/istio/blob/master/mixer/template/authorization/template.proto).

| Field        | Type                  | Description                                                  |
| ------------ | --------------------- | ------------------------------------------------------------ |
| `user`       | `string`              | Optional. The user name/ID that the subject represents.      |
| `group`      | `string`              | Optional. The group that the subject belongs to.             |
| `properties` | `map<string, string>` | Optional. The set of properties that identify the subject. In the above ServiceRoleBinding example, the second subject has two properties: service: “reviews” namespace: “abc” |